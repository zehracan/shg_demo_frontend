<template>
  <div class="container mt-4">
    <div class="card">
      <div class="card-header bg-primary text-white">
        <h4 class="mb-0">Göçmen Operasyon Formu</h4>
        <p class="mb-0">Tarih: {{ currentDate }}</p>
      </div>
      <div class="card-body">
        <form @submit.prevent="submitForm">
          <!-- İlk Satır -->
          <div class="row">
            <div class="col-md-6">
              <b-form-group label="Olay No" label-for="olay-no">
                <b-form-input
                  id="olay-no"
                  v-model="form.olayNo"
                  placeholder="Olay numarası giriniz..."
                ></b-form-input>
              </b-form-group>
            </div>
            
            <div class="col-md-6">
              <b-form-group label="Olay Tarihi" label-for="olay-tarihi">
                <b-form-datepicker
                  id="olay-tarihi"
                  v-model="form.olayTarihi"
                  :date-format-options="{ year: 'numeric', month: '2-digit', day: '2-digit' }"
                  locale="tr"
                ></b-form-datepicker>
              </b-form-group>
            </div>
          </div>

          <!-- İkinci Satır -->
          <div class="row">
            <div class="col-md-6">
              <b-form-group label="Bölge" label-for="bolge">
                <b-form-select
                  id="bolge"
                  v-model="form.bolge"
                  :options="bolgeOptions"
                >
                  <template #first>
                    <b-form-select-option :value="null" disabled>
                      Bölge seçiniz
                    </b-form-select-option>
                  </template>
                </b-form-select>
              </b-form-group>
            </div>
            
            <div class="col-md-6">
              <b-form-group label="Deniz" label-for="deniz">
                <b-form-select
                  id="deniz"
                  v-model="form.deniz"
                  :options="denizOptions"
                >
                  <template #first>
                    <b-form-select-option :value="null" disabled>
                      Deniz seçiniz
                    </b-form-select-option>
                  </template>
                </b-form-select>
              </b-form-group>
            </div>
          </div>

          <!-- Üçüncü Satır -->
          <div class="row">
            <div class="col-md-6">
              <b-form-group label="İl" label-for="il">
                <b-form-select
                  id="il"
                  v-model="form.il"
                  :options="ilOptions"
                  @change="onIlChange"
                >
                  <template #first>
                    <b-form-select-option :value="null" disabled>
                      İl seçiniz
                    </b-form-select-option>
                  </template>
                </b-form-select>
              </b-form-group>
            </div>
            
            <div class="col-md-6">
              <b-form-group label="İlçe" label-for="ilce">
                <b-form-select
                  id="ilce"
                  v-model="form.ilce"
                  :options="ilceOptions"
                  :disabled="!form.il"
                >
                  <template #first>
                    <b-form-select-option :value="null" disabled>
                      İlçe seçiniz
                    </b-form-select-option>
                  </template>
                </b-form-select>
              </b-form-group>
            </div>
          </div>

          
          <!-- Dördüncü Satır -->
          <div class="row">
            <div class="col-md-6">
              <b-form-group label="Enlem (DD MM SS)" label-for="enlem">
                <b-form-input
                  id="enlem"
                  v-model="form.enlem"
                  placeholder="Örn: 38 33 18"
                  pattern="[0-9]{2} [0-9]{2} [0-9]{2}"
                ></b-form-input>
                <small class="text-muted">Format: DD MM SS (örn: 38 33 18)</small>
              </b-form-group>
            </div>
            
            <div class="col-md-6">
              <b-form-group label="Boylam (DDD MM SS)" label-for="boylam">
                <b-form-input
                  id="boylam"
                  v-model="form.boylam"
                  placeholder="Örn: 026 18 42"
                  pattern="[0-9]{3} [0-9]{2} [0-9]{2}"
                ></b-form-input>
                <small class="text-muted">Format: DDD MM SS (örn: 026 18 42)</small>
              </b-form-group>
            </div>
          </div>
          <!-- Beşinci Satır -->
          <div class="row">
            <div class="col-md-6">
              <b-form-group label="Uyruk" label-for="uyruk">
                <b-form-select
                  id="uyruk"
                  v-model="form.uyruk"
                  :options="uyrukOptions"
                >
                  <template #first>
                    <b-form-select-option :value="null" disabled>
                      Uyruk seçiniz
                    </b-form-select-option>
                  </template>
                </b-form-select>
              </b-form-group>
            </div>
            
            <div class="col-md-6">
              <b-form-group label="Göçmen Sayısı" label-for="gocmen-sayisi">
                <b-form-input
                  id="gocmen-sayisi"
                  v-model="form.gocmenSayisi"
                  type="number"
                  min="0"
                  placeholder="Göçmen sayısı giriniz..."
                ></b-form-input>
              </b-form-group>
            </div>
          </div>

          <!-- Altıncı Satır -->
          <div class="row">
            <div class="col-md-6">
              <b-form-group label="Geçme Teşebbüs Yeri" label-for="gecme-tesebbüs">
                <b-form-select
                  id="gecme-tesebbüs"
                  v-model="form.gecmeTesebbusYer"
                  :options="gecmeTesebbusOptions"
                >
                  <template #first>
                    <b-form-select-option :value="null" disabled>
                      Teşebbüs yeri seçiniz
                    </b-form-select-option>
                  </template>
                </b-form-select>
              </b-form-group>
            </div>
            
            <div class="col-md-6">
              <b-form-group label="Vasıta İsmi" label-for="vasita-ismi">
                <b-form-select
                  id="vasita-ismi"
                  v-model="form.vasitaIsmi"
                  :options="vasitaIsmiOptions"
                >
                  <template #first>
                    <b-form-select-option :value="null" disabled>
                      Vasıta ismi seçiniz
                    </b-form-select-option>
                  </template>
                </b-form-select>
              </b-form-group>
            </div>
          </div>

          <!-- Yedinci Satır -->
          <div class="row">
            <div class="col-md-6">
              <b-form-group label="SG/Müşterek" label-for="sg-musterek">
                <b-form-radio-group
                  id="sg-musterek"
                  v-model="form.isSg"
                  :options="sgOptions"
                  buttons
                  button-variant="outline-primary"
                ></b-form-radio-group>
              </b-form-group>
            </div>
            
            <div class="col-md-6">
              <b-form-group label="Kullanılan Vasıta" label-for="kullanilan-vasita">
                <b-form-select
                  id="kullanilan-vasita"
                  v-model="form.kullanilanVasita"
                  :options="kullanilanVasitaOptions"
                >
                  <template #first>
                    <b-form-select-option :value="null" disabled>
                      Kullanılan vasıta seçiniz
                    </b-form-select-option>
                  </template>
                </b-form-select>
              </b-form-group>
            </div>
          </div>

          <!-- Butonlar -->
          <div class="row">
            <div class="col-md-12">
              <b-button
                type="submit"
                variant="primary"
                :disabled="isSubmitting"
                class="mr-2"
              >
                <b-spinner v-if="isSubmitting" small class="mr-1"></b-spinner>
                {{ isSubmitting ? 'Kaydediliyor...' : 'Kaydet' }}
              </b-button>
              <b-button
                type="button"
                variant="secondary"
                @click="resetForm"
              >
                Temizle
              </b-button>
            </div>
          </div>
        </form>
      </div>
    </div>

    <b-alert
      v-model="showAlert"
      :variant="alertVariant"
      dismissible
      class="mt-3"
    >
      {{ alertMessage }}
    </b-alert>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'GocmenOperasyonFormu',
  data() {
    return {
      form: {
        olayNo: '',
        olayTarihi: new Date().toISOString().split('T')[0],
        bolge: null,
        deniz: null,
        il: null,
        ilce: null,
        enlem: '',
        boylam: '',
        uyruk: null,
        gocmenSayisi: 0,
        gecmeTesebbusYer: null,
        vasitaIsmi: null,
        isSg: true,
        kullanilanVasita: null
      },
      isSubmitting: false,
      showAlert: false,
      alertMessage: '',
      alertVariant: 'success',
      ilceOptions: [],
      
      bolgeOptions: [
        { value: 'AKDENİZ', text: 'AKDENİZ' },
        { value: 'EGE', text: 'EGE' },
        { value: 'KARADENİZ', text: 'KARADENİZ' },
        { value: 'MARMARA', text: 'MARMARA' }
      ],
      
      denizOptions: [
        { value: 'AKDENİZ', text: 'AKDENİZ' },
        { value: 'EGE', text: 'EGE' },
        { value: 'VAN GÖLÜ', text: 'VAN GÖLÜ' }
      ],
      
      ilOptions: [
        { value: 'Adana', text: 'Adana' },
        { value: 'Adıyaman', text: 'Adıyaman' },
        { value: 'Afyonkarahisar', text: 'Afyonkarahisar' },
        { value: 'Ağrı', text: 'Ağrı' },
        { value: 'Amasya', text: 'Amasya' },
        { value: 'Ankara', text: 'Ankara' },
        { value: 'Antalya', text: 'Antalya' },
        { value: 'Artvin', text: 'Artvin' },
        { value: 'Aydın', text: 'Aydın' },
        { value: 'Balıkesir', text: 'Balıkesir' },
        { value: 'Bilecik', text: 'Bilecik' },
        { value: 'Bingöl', text: 'Bingöl' },
        { value: 'Bitlis', text: 'Bitlis' },
        { value: 'Bolu', text: 'Bolu' },
        { value: 'Burdur', text: 'Burdur' },
        { value: 'Bursa', text: 'Bursa' },
        { value: 'Çanakkale', text: 'Çanakkale' },
        { value: 'Çankırı', text: 'Çankırı' },
        { value: 'Çorum', text: 'Çorum' },
        { value: 'Denizli', text: 'Denizli' },
        { value: 'Diyarbakır', text: 'Diyarbakır' },
        { value: 'Düzce', text: 'Düzce' },
        { value: 'Edirne', text: 'Edirne' },
        { value: 'Elazığ', text: 'Elazığ' },
        { value: 'Erzincan', text: 'Erzincan' },
        { value: 'Erzurum', text: 'Erzurum' },
        { value: 'Eskişehir', text: 'Eskişehir' },
        { value: 'Gaziantep', text: 'Gaziantep' },
        { value: 'Giresun', text: 'Giresun' },
        { value: 'Gümüşhane', text: 'Gümüşhane' },
        { value: 'Hakkari', text: 'Hakkari' },
        { value: 'Hatay', text: 'Hatay' },
        { value: 'Iğdır', text: 'Iğdır' },
        { value: 'Isparta', text: 'Isparta' },
        { value: 'İstanbul', text: 'İstanbul' },
        { value: 'İzmir', text: 'İzmir' },
        { value: 'Kahramanmaraş', text: 'Kahramanmaraş' },
        { value: 'Karabük', text: 'Karabük' },
        { value: 'Karaman', text: 'Karaman' },
        { value: 'Kars', text: 'Kars' },
        { value: 'Kastamonu', text: 'Kastamonu' },
        { value: 'Kayseri', text: 'Kayseri' },
        { value: 'Kırıkkale', text: 'Kırıkkale' },
        { value: 'Kırklareli', text: 'Kırklareli' },
        { value: 'Kırşehir', text: 'Kırşehir' },
        { value: 'Kilis', text: 'Kilis' },
        { value: 'Kocaeli', text: 'Kocaeli' },
        { value: 'Konya', text: 'Konya' },
        { value: 'Kütahya', text: 'Kütahya' },
        { value: 'Malatya', text: 'Malatya' },
        { value: 'Manisa', text: 'Manisa' },
        { value: 'Mardin', text: 'Mardin' },
        { value: 'Mersin', text: 'Mersin' },
        { value: 'Muğla', text: 'Muğla' },
        { value: 'Muş', text: 'Muş' },
        { value: 'Nevşehir', text: 'Nevşehir' },
        { value: 'Niğde', text: 'Niğde' },
        { value: 'Ordu', text: 'Ordu' },
        { value: 'Osmaniye', text: 'Osmaniye' },
        { value: 'Rize', text: 'Rize' },
        { value: 'Sakarya', text: 'Sakarya' },
        { value: 'Samsun', text: 'Samsun' },
        { value: 'Siirt', text: 'Siirt' },
        { value: 'Sinop', text: 'Sinop' },
        { value: 'Sivas', text: 'Sivas' },
        { value: 'Şanlıurfa', text: 'Şanlıurfa' },
        { value: 'Şırnak', text: 'Şırnak' },
        { value: 'Tekirdağ', text: 'Tekirdağ' },
        { value: 'Tokat', text: 'Tokat' },
        { value: 'Trabzon', text: 'Trabzon' },
        { value: 'Tunceli', text: 'Tunceli' },
        { value: 'Uşak', text: 'Uşak' },
        { value: 'Van', text: 'Van' },
        { value: 'Yalova', text: 'Yalova' },
        { value: 'Yozgat', text: 'Yozgat' },
        { value: 'Zonguldak', text: 'Zonguldak' }
      ],
      
      uyrukOptions: [
        { value: 'suriye', text: 'Suriye' },
        { value: 'afganistan', text: 'Afganistan' },
        { value: 'pakistan', text: 'Pakistan' },
        { value: 'irak', text: 'Irak' },
        { value: 'fas', text: 'Fas' },
        { value: 'kamerun', text: 'Kamerun' },
        { value: 'nijerya', text: 'Nijerya' },
        { value: 'cad', text: 'Çad' },
        { value: 'tunus', text: 'Tunus' },
        { value: 'kenya', text: 'Kenya' },
        { value: 'zambiya', text: 'Zambiya' },
        { value: 'gana', text: 'Gana' },
        { value: 'benin', text: 'Benin' },
        { value: 'ozbekistan', text: 'Özbekistan' },
        { value: 'uganda', text: 'Uganda' },
        { value: 'etiyopya', text: 'Etiyopya' },
        { value: 'burkino_faso', text: 'Burkina Faso' },
        { value: 'kuveyt', text: 'Kuveyt' },
        { value: 'nepal', text: 'Nepal' },
        { value: 'togo', text: 'Togo' },
        { value: 'haiti', text: 'Haiti' },
        { value: 'liberya', text: 'Liberya' },
        { value: 'komor_adalari', text: 'Komor Adaları' },
        { value: 'urdun', text: 'Ürdün' },
        { value: 'moritanya', text: 'Moritanya' },
        { value: 'rusya', text: 'Rusya' },
        { value: 'azerbaycan', text: 'Azerbaycan' },
        { value: 'tanzanya', text: 'Tanzanya' },
        { value: 'nijer', text: 'Nijer' },
        { value: 'botsvana', text: 'Botsvana' }
      ],
      
      gecmeTesebbusOptions: [
        { value: 'BULAMAÇ', text: 'BULAMAÇ' },
        { value: 'DEDEAĞAÇ', text: 'DEDEAĞAÇ' },
        { value: 'KKTC', text: 'KKTC' },
        { value: 'LİMNİ', text: 'LİMNİ' },
        { value: 'MEİS', text: 'MEİS' },
        { value: 'MİDİLLİ', text: 'MİDİLLİ' },
        { value: 'RODOS', text: 'RODOS' },
        { value: 'SAKIZ', text: 'SAKIZ' },
        { value: 'SÖMBEKİ', text: 'SÖMBEKİ' },
        { value: 'SİSAM', text: 'SİSAM' },
        { value: 'VAN', text: 'VAN' },
        { value: 'İSTANKÖY', text: 'İSTANKÖY' },
        { value: 'İTALYA', text: 'İTALYA' }
      ],
      
      vasitaIsmiOptions: [
        { value: '2 CAN SALI', text: '2 CAN SALI' },
        { value: '3 CAN SALI', text: '3 CAN SALI' },
        { value: '4 CAN SALI', text: '4 CAN SALI' },
        { value: 'ADA-34', text: 'ADA-34' },
        { value: 'ADA/KARA ÜZERİNDEN', text: 'ADA/KARA ÜZERİNDEN' },
        { value: 'ASTERİ', text: 'ASTERİ' },
        { value: 'ASİL TÜRK', text: 'ASİL TÜRK' },
        { value: 'ATABEY-35', text: 'ATABEY-35' },
        { value: 'BODOTO 10', text: 'BODOTO 10' },
        { value: 'BİR MASAL', text: 'BİR MASAL' },
        { value: 'CAPTAIN ZÜLFÜ', text: 'CAPTAIN ZÜLFÜ' },
        { value: 'ÇEPELİ', text: 'ÇEPELİ' },
        { value: 'DEDEAĞAÇ', text: 'DEDEAĞAÇ' },
        { value: 'DOLPHİ', text: 'DOLPHİ' },
        { value: 'ELİF', text: 'ELİF' },
        { value: 'EMİNE', text: 'EMİNE' },
        { value: 'ERAY', text: 'ERAY' },
        { value: 'FENİKS', text: 'FENİKS' },
        { value: 'FRİXOS', text: 'FRİXOS' },
        { value: 'GOOD MOON', text: 'GOOD MOON' },
        { value: 'KARATAŞ 1', text: 'KARATAŞ 1' },
        { value: 'KAZAN-1', text: 'KAZAN-1' },
        { value: 'KIBRIS', text: 'KIBRIS' },
        { value: 'KOTİL', text: 'KOTİL' },
        { value: 'LİMON-3', text: 'LİMON-3' },
        { value: 'MESUT REİS', text: 'MESUT REİS' },
        { value: 'MUĞLA-1', text: 'MUĞLA-1' },
        { value: 'MUSTAFA', text: 'MUSTAFA' },
        { value: 'NURAY', text: 'NURAY' },
        { value: 'ONUR', text: 'ONUR' },
        { value: 'PARSS-1 VE ADA 34', text: 'PARSS-1 VE ADA 34' },
        { value: 'RACİNG 1', text: 'RACİNG 1' },
        { value: 'RAGNAR-47', text: 'RAGNAR-47' },
        { value: 'RONYA', text: 'RONYA' },
        { value: 'RÜZGAR REİS 01', text: 'RÜZGAR REİS 01' },
        { value: 'S. AMAZON 35', text: 'S. AMAZON 35' },
        { value: 'SKY 48', text: 'SKY 48' },
        { value: 'SUVAARİ', text: 'SUVAARİ' },
        { value: 'ŞAHİN-1', text: 'ŞAHİN-1' },
        { value: 'T.C.G.S. 902', text: 'T.C.G.S. 902' },
        { value: 'T.C.S.G. 311', text: 'T.C.S.G. 311' },
        { value: 'TRİTON', text: 'TRİTON' },
        { value: 'UMBRA', text: 'UMBRA' },
        { value: 'VENESUELLA', text: 'VENESUELLA' },
        { value: 'WHİTE GREAM', text: 'WHİTE GREAM' },
        { value: 'YEDİTEPE-1', text: 'YEDİTEPE-1' },
        { value: 'YUSUF REİS-3', text: 'YUSUF REİS-3' },
        { value: 'YÜZEREK', text: 'YÜZEREK' },
        { value: 'İSİMSİZ', text: 'İSİMSİZ' },
        { value: 'ŞİŞME YATAK', text: 'ŞİŞME YATAK' }
      ],
      
      kullanilanVasitaOptions: [
        { value: 'ADA/KARA ÜZERİNDEN', text: 'ADA/KARA ÜZERİNDEN' },
        { value: 'AHŞAP TEKNE', text: 'AHŞAP TEKNE' },
        { value: 'BALIK AVLAMA TEKNESİ', text: 'BALIK AVLAMA TEKNESİ' },
        { value: 'BALIKÇI', text: 'BALIKÇI' },
        { value: 'CAN SALI', text: 'CAN SALI' },
        { value: 'DENİZ ÜZERİNDEN', text: 'DENİZ ÜZERİNDEN' },
        { value: 'FERİBOT', text: 'FERİBOT' },
        { value: 'FİBER KARİNALI LASTİK BOT', text: 'FİBER KARİNALI LASTİK BOT' },
        { value: 'FİBER TEKNE', text: 'FİBER TEKNE' },
        { value: 'GÖÇMEN BOT', text: 'GÖÇMEN BOT' },
        { value: 'JANDARMA BOT', text: 'JANDARMA BOT' },
        { value: 'LASTİK BOT', text: 'LASTİK BOT' },
        { value: 'MASAÜSTÜ BOT', text: 'MASAÜSTÜ BOT' },
        { value: 'MOTORLU ZODYAK', text: 'MOTORLU ZODYAK' },
        { value: 'RIB BOT', text: 'RIB BOT' },
        { value: 'SAHİL GÜVENLİK BOT', text: 'SAHİL GÜVENLİK BOT' },
        { value: 'SAVUNMA BOT', text: 'SAVUNMA BOT' }
      ],
      
      sgOptions: [
        { text: 'SG', value: true },
        { text: 'Müşterek', value: false }
      ]
    };
  },
  computed: {
    currentDate() {
      return new Date().toLocaleDateString('tr-TR', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      });
    }
  },
  methods: {
    onIlChange() {
      this.form.ilce = null;
      this.loadIlceOptions(this.form.il);
    },
    
    async loadIlceOptions(il) {
      try {
        const response = await axios.get(`/api/GocmenOperasyon/ilceler/${il}`);
        if (response.data.success) {
          this.ilceOptions = response.data.data;
        } else {
          this.ilceOptions = [];
        }
      } catch (error) {
        console.error('İlçe yükleme hatası:', error);
        this.ilceOptions = [];
      }
    },
    
    async submitForm() {
      this.isSubmitting = true;
      this.showAlert = false;

      try {
        if (!this.form.olayNo || !this.form.bolge || !this.form.deniz || !this.form.il || !this.form.ilce || !this.form.uyruk || !this.form.gecmeTesebbusYer) {
          this.showAlertMessage('Lütfen tüm zorunlu alanları doldurunuz.', 'danger');
          return;
        }

        const response = await axios.post('/api/GocmenOperasyon', {
          olayNo: this.form.olayNo,
          olayTarihi: new Date(this.formData.olayTarihi).toISOString(),
          bolge: this.form.bolge,
          deniz: this.form.deniz,
          il: this.form.il,
          ilce: this.form.ilce,
          enlem: this.form.enlem,
          boylam: this.form.boylam,
          uyruk: this.form.uyruk,
          gocmenSayisi: parseInt(this.form.gocmenSayisi) || 0,
          gecmeTesebbusYer: this.form.gecmeTesebbusYer,
          vasitaIsmi: this.form.vasitaIsmi,
          isSg: this.form.isSg,
          kullanilanVasita: this.form.kullanilanVasita
        });

        if (response.data.success) {
          this.showAlertMessage('Operasyon verisi başarıyla kaydedildi!', 'success');
          this.resetForm();
        } else {
          this.showAlertMessage('Kayıt sırasında bir hata oluştu.', 'danger');
        }
      } catch (error) {
        console.error('Form submission error:', error);
        this.showAlertMessage('Sunucu hatası oluştu. Lütfen tekrar deneyiniz.', 'danger');
      } finally {
        this.isSubmitting = false;
      }
    },
    
    resetForm() {
      this.form = {
        olayNo: '',
        olayTarihi: new Date().toISOString().split('T')[0],
        bolge: null,
        deniz: null,
        il: null,
        ilce: null,
        enlem: null,
        boylam: null,
        uyruk: null,
        gocmenSayisi: 0,
        gecmeTesebbusYer: null,
        vasitaIsmi: null,
        isSg: true,
        kullanilanVasita: null
      };
      this.ilceOptions = [];
    },
    
    showAlertMessage(message, variant) {
      this.alertMessage = message;
      this.alertVariant = variant;
      this.showAlert = true;
    }
  }
};
</script>

<style scoped>
.card {
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.card-header {
  border-bottom: 2px solid #dee2e6;
}

.form-group {
  margin-bottom: 1.5rem;
}
</style>