<template>
  <div class="container mt-4">
    <div class="card">
      <div class="card-header bg-info text-white">
        <h4>Operasyon Raporu</h4>
      </div>
      <div class="card-body">

        <!-- Özet Bilgiler ve Tablo -->
        <div v-if="isLoading" class="text-center">
          <b-spinner label="Yükleniyor..."></b-spinner>
          <p>Rapor verileri yükleniyor...</p>
        </div>
        <div v-else>
          <div class="alert alert-success">
            <strong>{{ reportItems.length }}</strong> adet onaylanmış operasyon bulundu.
          </div>
          <b-table
            striped
            hover
            responsive
            :items="reportItems"
            :fields="reportFields"
            empty-text="Gösterilecek kayıt bulunamadı."
          >
          </b-table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'OperasyonRaporu',
  data() {
    return {
      isLoading: false,
      reportItems: [],
      reportFields: [
        { key: 'olayNo', label: 'Olay No', sortable: true },
        {
          key: 'olayTarihi',
          label: 'Olay Tarihi',
          sortable: true,
          formatter: (value) => {
            const date = new Date(value);
            return date.toLocaleString('tr-TR', {
              year: 'numeric', month: 'long', day: 'numeric', hour: '2-digit', minute: '2-digit'
            });
          }
        },
        { key: 'bolge', label: 'Bölge', sortable: true },
        { key: 'il', label: 'İl', sortable: true },
        { key: 'toplamGocmen', label: 'Toplam Göçmen Sayısı', sortable: true, class: 'text-right' },
        { key: 'uyruk', label: 'Uyruk', sortable: true },
        { key: 'kategori', label: 'Olay Kategorisi', sortable: true },
      ],
    };
  },
  created() {
    this.fetchReportData();
  },
  methods: {
    async fetchReportData() {
      this.isLoading = true;
      try {
        const response = await axios.get('/api/GocmenOperasyon/rapor');
        this.reportItems = response.data;
      } catch (error) {
        console.error("Rapor verileri çekilemedi:", error);
        alert('Rapor verileri yüklenirken bir hata oluştu.');
      } finally {
        this.isLoading = false;
      }
    },
  },
};
</script>