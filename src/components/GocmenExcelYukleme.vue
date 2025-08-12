<template>
  <div class="container mt-4">
    <div class="card">
      <div class="card-header bg-success text-white">
        <h4 class="mb-0">Operasyon Verisi Toplu Yükleme</h4>
        <p class="mb-0">Excel dosyası ile toplu veri yükleme</p>
      </div>
      <div class="card-body">
        <!-- Dosya Seçme -->
        <div class="row mb-3">
          <div class="col-md-12">
            <b-form-group label="Excel Dosyası Seçin" label-for="file-input">
              <b-form-file
                id="file-input"
                v-model="selectedFile"
                accept=".xlsx,.xls"
                placeholder="Excel dosyası seçiniz..."
                drop-placeholder="Excel dosyasını buraya bırakın..."
                @input="onFileChange"
              ></b-form-file>
            </b-form-group>
          </div>
        </div>

        <!-- Excel Önizleme -->
        <div v-if="excelData.length > 0" class="mb-3">
          <h5>Dosya Önizleme:</h5>
          <b-table 
            :items="excelData.slice(0, 5)" 
            :fields="excelFields"
            striped 
            hover 
            small
            responsive
            class="mt-2"
          >
          </b-table>
          <p class="text-muted">Toplam {{ excelData.length }} kayıt bulundu (İlk 5 kayıt gösteriliyor)</p>
        </div>

        <!-- Yükleme Butonu -->
        <div class="row">
          <div class="col-md-12">
            <b-button
              variant="success"
              :disabled="excelData.length === 0 || isUploading"
              @click="uploadData"
              class="mr-2"
            >
              <b-spinner v-if="isUploading" small class="mr-1"></b-spinner>
              {{ isUploading ? 'Yükleniyor...' : 'Verileri Yükle' }}
            </b-button>
            <b-button
              variant="secondary"
              @click="clearData"
            >
              Temizle
            </b-button>
          </div>
        </div>

        <!-- Yükleme Sonucu -->
        <div v-if="uploadResult" class="mt-3">
          <b-alert :variant="uploadResult.success ? 'success' : 'danger'" show dismissible>
            <h6 v-if="uploadResult.success" class="mb-2">
              <i class="fas fa-check-circle"></i> Başarılı!
            </h6>
            <h6 v-else class="mb-2">
              <i class="fas fa-exclamation-triangle"></i> Hata!
            </h6>
            {{ uploadResult.message }}
            <div v-if="uploadResult.details" class="mt-2">
              <small>{{ uploadResult.details }}</small>
            </div>
            <div v-if="uploadResult.success && uploadResult.count" class="mt-2">
              <strong>Toplam {{ uploadResult.count }} kayıt başarıyla veritabanına kaydedildi.</strong>
            </div>
          </b-alert>
        </div>
      </div>
    </div>

    <!-- Excel Format Bilgisi -->
    <div class="card mt-4">
      <div class="card-header bg-info text-white">
        <h5 class="mb-0">Excel Dosya Formatı</h5>
      </div>
      <div class="card-body">
        <p>Excel dosyanızın ilk satırında aşağıdaki sütun başlıkları olmalıdır:</p>
        <b-table 
          :items="formatExample" 
          :fields="formatFieldsDisplay"
          striped 
          small
          responsive
          class="mt-2"
          style="font-size: 0.8rem;"
        ></b-table>
        <div class="mt-3">
          <h6>Önemli Notlar:</h6>
          <ul class="text-muted">
            <li><strong>olay_tarihi:</strong> YYYY-MM-DD formatında (örn: 2025-07-18)</li>
            <li><strong>enlem/boylam:</strong> DD MM SS formatında (örn: 38 33 18 / 026 18 42)</li>
            <li><strong>gocmen_sayisi:</strong> Tam sayı (örn: 15)</li>
            <li><strong>is_sg:</strong> true (SG için) veya false (Müşterek için)</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as XLSX from 'xlsx';
import axios from 'axios';

export default {
  name: 'GocmenExcelYukleme',
  data() {
    return {
      selectedFile: null,
      excelData: [],
      excelFields: [],
      isUploading: false,
      uploadResult: null,
      formatFields: [
        { key: 'olay_no', label: 'olay_no' },
        { key: 'olay_tarihi', label: 'olay_tarihi' },
        { key: 'bolge', label: 'bolge' },
        { key: 'deniz', label: 'deniz' },
        { key: 'il', label: 'il' },
        { key: 'ilce', label: 'ilce' },
        { key: 'enlem', label: 'enlem' },
        { key: 'boylam', label: 'boylam' },
        { key: 'uyruk', label: 'uyruk' },
        { key: 'gocmen_sayisi', label: 'gocmen_sayisi' },
        { key: 'gecme_tesebbüs_yer', label: 'gecme_tesebbüs_yer' },
        { key: 'vasita_ismi', label: 'vasita_ismi' },
        { key: 'is_sg', label: 'is_sg' },
        { key: 'kullanilan_vasita', label: 'kullanilan_vasita' }
      ],
      formatFieldsDisplay: [
        { key: 'olay_no', label: 'olay_no' },
        { key: 'olay_tarihi', label: 'olay_tarihi' },
        { key: 'bolge', label: 'bolge' },
        { key: 'deniz', label: 'deniz' },
        { key: 'il', label: 'il' },
        { key: 'ilce', label: 'ilce' },
        { key: 'enlem', label: 'enlem' },
        { key: 'boylam', label: 'boylam' },
        { key: 'uyruk', label: 'uyruk' },
        { key: 'gocmen_sayisi', label: 'gocmen_sayisi' },
        { key: 'gecme_tesebbüs_yer', label: 'gecme_tesebbüs_yer' },
        { key: 'vasita_ismi', label: 'vasita_ismi' },
        { key: 'is_sg', label: 'is_sg' },
        { key: 'kullanilan_vasita', label: 'kullanilan_vasita' }
      ],
      formatExample: [
        {
          olay_no: '2025-001',
          olay_tarihi: '2025-07-18',
          bolge: 'EGE',
          deniz: 'EGE',
          il: 'İzmir',
          ilce: 'Çeşme',
          enlem: '38 33 18',
          boylam: '026 18 42',
          uyruk: 'suriye',
          gocmen_sayisi: 15,
          'gecme_tesebbüs_yer': 'MİDİLLİ',
          vasita_ismi: 'KAZAN-1',
          is_sg: true,
          kullanilan_vasita: 'LASTİK BOT'
        }
      ]
    };
  },
  methods: {
    onFileChange(file) {
      console.log('File change event:', file);
      
      if (file) {
        console.log('File name:', file.name);
        console.log('File type:', file.type);
        this.readExcelFile(file);
      } else {
        console.log('No file selected');
      }
    },
    
    readExcelFile(file) {
      const reader = new FileReader();
      reader.onload = (e) => {
        try {
          const data = new Uint8Array(e.target.result);
          const workbook = XLSX.read(data, { type: 'array' });
          const firstSheetName = workbook.SheetNames[0];
          const worksheet = workbook.Sheets[firstSheetName];
          
          const jsonData = XLSX.utils.sheet_to_json(worksheet, { 
            header: 1,
            defval: ''
          });
          
          console.log('Excel verileri:', jsonData);
          
          if (jsonData.length > 1) {
            const headers = jsonData[0].map(h => String(h).toLowerCase().trim().replace(/ /g, '_'));
            const rows = jsonData.slice(1).filter(row => 
              row.some(cell => cell !== null && cell !== undefined && cell !== '')
            );
            
            console.log('Headers:', headers);
            console.log('Rows:', rows);
            
            this.excelFields = headers.map(header => ({
              key: header,
              label: header.charAt(0).toUpperCase() + header.slice(1).replace(/_/g, ' ')
            }));
            
            this.excelData = rows.map(row => {
              const obj = {};
              headers.forEach((header, index) => {
                obj[header] = row[index] || '';
              });
              return obj;
            });
            
            console.log('Excel Data:', this.excelData);
          } else {
            console.error('Excel dosyasında yeterli veri bulunamadı');
            alert('Excel dosyasında yeterli veri bulunamadı!');
          }
        } catch (error) {
          console.error('Excel okuma hatası:', error);
          alert('Excel dosyası okunamadı! Hata: ' + error.message);
        }
      };
      reader.readAsArrayBuffer(file);
    },
    
    async uploadData() {
      this.isUploading = true;
      this.uploadResult = null;
      
      try {
        const dataToSend = this.excelData.map(row => {
          // Tarih formatını düzelt
          let olayTarihi = row.olay_tarihi;
          if (olayTarihi && typeof olayTarihi === 'number') {
            // Excel serial date'i normal tarihe çevir
            const date = new Date((olayTarihi - 25569) * 86400 * 1000);
            olayTarihi = date.toISOString().split('T')[0];
          } else if (olayTarihi && typeof olayTarihi === 'string') {
            // String tarih formatını kontrol et
            const date = new Date(olayTarihi);
            if (!isNaN(date.getTime())) {
              olayTarihi = date.toISOString().split('T')[0];
            } else {
              olayTarihi = new Date().toISOString().split('T')[0];
            }
          } else {
            olayTarihi = new Date().toISOString().split('T')[0];
          }

          // is_sg boolean değerine çevir
          let isSg = true;
          if (row.is_sg !== undefined && row.is_sg !== null) {
            if (typeof row.is_sg === 'string') {
              isSg = row.is_sg.toLowerCase() === 'true' || row.is_sg.toLowerCase() === 'sg';
            } else {
              isSg = Boolean(row.is_sg);
            }
          }

          return {
            olayNo: row.olay_no || '',
            olayTarihi: olayTarihi,
            bolge: row.bolge || '',
            deniz: row.deniz || '',
            il: row.il || '',
            ilce: row.ilce || '',
            enlem: row.enlem ? parseFloat(row.enlem) : null,
            boylam: row.boylam ? parseFloat(row.boylam) : null,
            uyruk: row.uyruk || '',
            gocmenSayisi: parseInt(row.gocmen_sayisi) || 0,
            gecmeTesebbusYer: row['gecme_tesebbüs_yer'] || row.gecme_tesebbüs_yer || '',
            vasitaIsmi: row.vasita_ismi || null,
            isSg: isSg,
            kullanilanVasita: row.kullanilan_vasita || null
          };
        });

        console.log('Gönderilecek veri:', dataToSend);

        const response = await axios.post('/api/GocmenOperasyon/bulk', dataToSend);

        if (response.data.success) {
          this.uploadResult = {
            success: true,
            message: 'Excel dosyası başarıyla işlendi ve veriler kaydedildi!',
            count: this.excelData.length
          };
          console.log('Upload result set to:', this.uploadResult);
          this.clearData();
          
          // 5 saniye sonra mesajı temizle
          setTimeout(() => {
            this.uploadResult = null;
          }, 5000);
        } else {
          this.uploadResult = {
            success: false,
            message: response.data.message || 'Yükleme başarısız'
          };
          console.log('Upload failed, result:', this.uploadResult);
        }
      } catch (error) {
        console.error('Upload error:', error);
        this.uploadResult = {
          success: false,
          message: 'Yükleme sırasında hata oluştu',
          details: error.response?.data?.message || error.message
        };
      } finally {
        this.isUploading = false;
      }
    },
    
    clearData() {
      this.selectedFile = null;
      this.excelData = [];
      this.excelFields = [];
      console.log('clearData called, uploadResult:', this.uploadResult);
    }
  }
};
</script>

<style scoped>
.card {
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 1.5rem;
}

ul {
  margin-bottom: 0;
}

li {
  margin-bottom: 0.5rem;
}
</style>