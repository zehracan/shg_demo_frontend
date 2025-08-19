<template>
  <div class="onay-container">
    <!-- 1. Sekme Başlıkları -->
    <div class="tabs">
      <button @click="changeTab('Pending')" :class="{ active: currentTab === 'Pending' }">
        Bekleyen Onaylar ({{ items.Pending.length }})
      </button>
      <button @click="changeTab('Approved')" :class="{ active: currentTab === 'Approved' }">
        Onaylanmış Operasyonlar
      </button>
      <button @click="changeTab('Rejected')" :class="{ active: currentTab === 'Rejected' }">
        Reddedilmiş Operasyonlar
      </button>
    </div>

    <!-- 2. Sekme İçerikleri -->
    <div class="tab-content">
      <div v-if="isLoading" class="loading-state"><p>Veriler yükleniyor, lütfen bekleyin...</p></div>

      <!-- Bekleyenler Sekmesi -->
      <div v-else-if="currentTab === 'Pending'">
        <div v-if="!items.Pending.length" class="empty-state">Onay bekleyen operasyon bulunmuyor.</div>
        <div v-else class="pending-layout">
          <div class="item-list">
            <ul>
              <li v-for="item in items.Pending" :key="item.id" @click="selectItem(item)" :class="{ selected: selectedItem && selectedItem.id === item.id }">
                <strong>{{ item.olayNo || 'Olay No Yok' }}</strong>
                <span>Kayıt Tarihi: {{ formatDate(item.olayTarihi) }}</span>
              </li>
            </ul>
          </div>
          <div class="item-detail">
            <div v-if="!selectedItem" class="placeholder">Detayları görmek için lütfen soldaki listeden bir operasyon seçin.</div>
            <div v-else>
              <GocmenOperasyonFormu
                :initial-data="selectedItem"
                :is-disabled="true"
              />
              <div class="actions">
                <button @click="updateStatus(selectedItem.olayId, 'Approved')" class="btn-approve">ONAYLA</button>
                <button @click="updateStatus(selectedItem.olayId, 'Rejected')" class="btn-reject">REDDET</button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Diğer Sekmeler -->
      <div v-else>
        <div v-if="!currentItems.length" class="empty-state">Bu kategoride gösterilecek kayıt bulunmuyor.</div>
        <table v-else class="list-table">
          <thead>
            <tr>
              <th>Olay No</th>
              <th>İşlem Tarihi</th>
              <th>Durum</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in currentItems" :key="item.olayId">
              <td>{{ item.olayNo }}</td>
              <td>{{ formatDate(item.olayTarihi) }}</td>
              <td><span :class="`status-${item.status.toLowerCase()}`">{{ item.status }}</span></td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import GocmenOperasyonFormu from './GocmenOperasyonFormu.vue'; 

export default {
  name: 'OperasyonOnayListesi',
  components: {
    GocmenOperasyonFormu
  },
  data() {
    return {
      currentTab: 'Pending',
      items: { Pending: [], Approved: [], Rejected: [] },
      selectedItem: null,
      isLoading: false
    };
  },
  computed: {
    currentItems() {
      return this.items[this.currentTab] || [];
    }
  },
  created() {
    this.fetchDataForCurrentTab();
  },
  methods: {
    changeTab(tabName) {
      this.currentTab = tabName;
      this.selectedItem = null;
      this.fetchDataForCurrentTab();
    },
    async fetchDataForCurrentTab() {
      this.isLoading = true;
      try {
        const response = await axios.get(`/api/GocmenOperasyon?status=${this.currentTab}`);
        this.items[this.currentTab] = response.data;
      } catch (error) {
        console.error(`${this.currentTab} verileri çekilirken hata:`, error);
        alert('Veriler yüklenirken bir hata oluştu.');
      } finally {
        this.isLoading = false;
      }
    },
    selectItem(item) {
      this.selectedItem = item;
    },
    async updateStatus(itemId, newStatus) {
      const actionText = newStatus === 'Approved' ? 'onaylamak' : 'reddetmek';
      if (!confirm(`Bu operasyonu ${actionText} istediğinizden emin misiniz?`)) { return; }
      try {
        await axios.patch(`/api/GocmenOperasyon/${itemId}/status`, { status: newStatus });

        this.items.Pending = this.items.Pending.filter(item => item.olayId !== itemId);
        this.selectedItem = null;
        this.items[newStatus] = []; 
        alert('Operasyon durumu başarıyla güncellendi!');
      } catch (error) {
        console.error('Durum güncellenirken hata:', error);
        alert('İşlem sırasında bir hata oluştu.');
      }
    },
    formatDate(dateString) {
      if (!dateString) return '';
      const options = { year: 'numeric', month: 'long', day: 'numeric' };
      return new Date(dateString).toLocaleDateString('tr-TR', options);
    }
  }
}
</script>

<style scoped>
/* Genel Konteyner */
.onay-container {
  border: 1px solid #dfe4ea;
  border-radius: 8px;
  overflow: hidden;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  background-color: #fff;
}

/* Sekmeler */
.tabs {
  display: flex;
  background-color: #f1f3f5;
  border-bottom: 1px solid #dfe4ea;
}
.tabs button {
  padding: 15px 25px;
  border: none;
  background-color: transparent;
  cursor: pointer;
  font-size: 16px;
  color: #495057;
  border-bottom: 3px solid transparent;
  transition: all 0.2s ease-in-out;
  position: relative;
  top: 1px;
}
.tabs button:hover {
  background-color: #e9ecef;
}
.tabs button.active {
  font-weight: 600;
  color: #1971c2;
  border-bottom-color: #1971c2;
  background-color: #fff;
}

/* Sekme İçeriği ve Durum Metinleri */
.tab-content {
  padding: 20px;
}
.loading-state, .empty-state {
  text-align: center;
  padding: 50px 20px;
  color: #868e96;
  font-size: 1.1em;
}

/* Bekleyenler Sekmesi Düzeni */
.pending-layout {
  display: flex;
  gap: 20px;
  min-height: 65vh;
}
.item-list {
  width: 35%;
  border-right: 1px solid #e9ecef;
  padding-right: 20px;
  max-height: 70vh;
  overflow-y: auto;
}
.item-list ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
}
.item-list li {
  padding: 12px;
  border-radius: 6px;
  cursor: pointer;
  margin-bottom: 8px;
  display: flex;
  flex-direction: column;
  border: 1px solid #dee2e6;
  transition: background-color 0.2s, border-color 0.2s;
}
.item-list li:hover {
  background-color: #f8f9fa;
  border-color: #adb5bd;
}
.item-list li.selected {
  background-color: #e7f5ff;
  border-color: #74c0fc;
  color: #1864ab;
}
.item-list li strong {
  font-size: 1em;
  font-weight: 600;
  margin-bottom: 4px;
}
.item-list li span {
  font-size: 0.85em;
  color: #868e96;
}
.item-list li.selected span {
  color: #1864ab;
}

.item-detail {
  width: 65%;
  max-height: 75vh;
  overflow-y: auto;
  padding-right: 10px;
}
.placeholder {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  color: #adb5bd;
  font-size: 1.2em;
  text-align: center;
}

/* Onay/Red Butonları */
.actions {
  margin-top: 20px;
  display: flex;
  gap: 10px;
  position: sticky;
  bottom: -20px; 
  background: white;
  padding: 15px 0;
  border-top: 1px solid #e9ecef;
}
.actions button {
  flex-grow: 1;
  padding: 12px 24px;
  border: none;
  border-radius: 6px;
  color: white;
  cursor: pointer;
  font-size: 16px;
  font-weight: bold;
  transition: opacity 0.2s;
}
.actions button:hover {
  opacity: 0.85;
}
.btn-approve {
  background-color: #2f9e44 !important; 
}
.btn-reject {
  background-color: #e03131 !important; 
}

/* Liste Tablosu (Onaylanan/Reddedilen) */
.list-table {
  width: 100%;
  border-collapse: collapse;
}
.list-table th, .list-table td {
  padding: 12px 15px;
  text-align: left;
  border-bottom: 1px solid #dee2e6;
}
.list-table th {
  background-color: #f8f9fa;
  font-weight: 600;
  color: #495057;
  font-size: 0.9em;
  text-transform: uppercase;
}
.status-approved {
  color: #2f9e44;
  font-weight: bold;
}
.status-rejected {
  color: #e03131;
  font-weight: bold;
}
</style>