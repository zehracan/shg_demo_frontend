<template>
  <div class="container mt-4">
    
    <b-modal id="status-modal" :title="modalTitle" :header-bg-variant="modalVariant" header-text-variant="white" ok-only>
      <p class="my-4">{{ modalMessage }}</p>
    </b-modal>

    <div class="card">
      <div class="card-header bg-primary text-white">
        <h4 class="mb-0">{{ isDisabled ? 'Göçmen Operasyon Detayları' : 'Göçmen Operasyon Formu' }}</h4>
        <p class="mb-0">Tarih: {{ currentDate }}</p>
      </div>
      <div class="card-body">
        <form @submit.prevent="isDisabled ? null : submitForm()">
          
          <b-tabs v-model="tabIndex" content-class="mt-3" fill @activate-tab="handleTabActivation">
            <!-- TAB 1: ANA OLAY BİLGİLERİ -->
            <b-tab title="1. Ana Olay">
              <fieldset class="border p-3 rounded">
                <legend class="w-auto h5 text-primary">Ana Olay Bilgileri</legend>
                <div class="row">
                  <div class="col-md-4"><b-form-group label="Olay No" label-for="olay-no"><b-form-input id="olay-no" v-model="form.olayNo" placeholder="Olay numarası giriniz..." required :disabled="isDisabled"></b-form-input></b-form-group></div>
                  <div class="col-md-4"><b-form-group label="Olay Tarihi" label-for="olay-tarihi"><b-form-datepicker id="olay-tarihi" v-model="form.olayTarihi" locale="tr" required :disabled="isDisabled"></b-form-datepicker></b-form-group></div>
                  <div class="col-md-4"><b-form-group label="Olay Saati" label-for="olay-saati"><b-form-timepicker id="olay-saati" v-model="form.olaySaati" locale="tr" required :disabled="isDisabled"></b-form-timepicker></b-form-group></div>
                </div>
                <div class="row">
                  <div class="col-md-4"><b-form-group label="Bölge" label-for="bolge"><b-form-select id="bolge" v-model="form.bolge" :options="bolgeOptions" required :disabled="isDisabled"></b-form-select></b-form-group></div>
                  <div class="col-md-4"><b-form-group label="Deniz" label-for="deniz"><b-form-select id="deniz" v-model="form.deniz" :options="denizOptions" required :disabled="isDisabled"></b-form-select></b-form-group></div>
                  <div class="col-md-4"><b-form-group label="Geçme Teşebbüs Yeri" label-for="yer"><b-form-select id="yer" v-model="form.yer" :options="gecmeTesebbusOptions" required :disabled="isDisabled"><template #first><b-form-select-option :value="null" disabled>Teşebbüs yeri seçiniz</b-form-select-option></template></b-form-select></b-form-group></div>
                </div>
                <div class="row">
                  <div class="col-md-4"><b-form-group label="SG/Müşterek" label-for="sg-mi"><b-form-radio-group v-model="form.sgMi" :options="sgOptions" buttons button-variant="outline-primary" class="w-100" required :disabled="isDisabled"></b-form-radio-group></b-form-group></div>
                  <div class="col-md-4"><b-form-group label="Kategori" label-for="olay-kategorisi"><b-form-select v-model="form.olayKategorisi" :options="olayKategoriOptions" required :disabled="isDisabled"></b-form-select></b-form-group></div>
                  <div class="col-md-4"><b-form-group label="Denize Atma" label-for="denize-atma"><b-form-select v-model="form.denizeAtma" :options="varYokOptions" required :disabled="isDisabled"></b-form-select></b-form-group></div>
                </div>
                <div class="row">
                  <div class="col-md-6"><b-form-group label="İl" label-for="il"><b-form-select id="il" v-model="form.il" :options="ilOptions" @change="onIlChange" required :disabled="isDisabled"><template #first><b-form-select-option :value="null" disabled>İl seçiniz</b-form-select-option></template></b-form-select></b-form-group></div>
                  <div class="col-md-6"><b-form-group label="İlçe" label-for="ilce"><b-form-select id="ilce" v-model="form.ilce" :options="ilceOptions" :disabled="!form.il || isDisabled" required><template #first><b-form-select-option :value="null" disabled>İlçe seçiniz</b-form-select-option></template></b-form-select></b-form-group></div>
                </div>
                <div class="row">
                  <div class="col-md-6"><b-form-group label="Enlem (DD MM SS)" label-for="enlem"><b-form-input id="enlem" v-model="form.enlem" placeholder="Örn: 38 33 18" pattern="[0-9]{2} [0-9]{2} [0-9]{2}" required :disabled="isDisabled"></b-form-input><small class="text-muted">Format: DD MM SS</small></b-form-group></div>
                  <div class="col-md-6"><b-form-group label="Boylam (DDD MM SS)" label-for="boylam"><b-form-input id="boylam" v-model="form.boylam" placeholder="Örn: 026 18 42" pattern="[0-9]{3} [0-9]{2} [0-9]{2}" required :disabled="isDisabled"></b-form-input><small class="text-muted">Format: DDD MM SS</small></b-form-group></div>
                </div>
              </fieldset>
              <div v-if="!isDisabled" class="d-flex justify-content-end mt-3">
                <b-button variant="primary" @click="nextTab">İleri</b-button>
              </div>
            </b-tab>

            <!-- TAB 2: UYRUK BİLGİLERİ -->
            <b-tab title="2. Uyruklar">
               <fieldset class="border p-3 rounded">
                <legend class="w-auto h5 text-primary">Uyruk ve Sayıları *</legend>
                <div v-if="!isDisabled" class="row align-items-end p-2 mb-2 bg-light border rounded">
                  <div class="col-md-6"><b-form-group label-class="font-weight-bold" label="Uyruk Ekle"><b-form-select v-model="yeniUyruk.uyruk" :options="uyrukOptions"></b-form-select></b-form-group></div>
                  <div class="col-md-4"><b-form-group label-class="font-weight-bold" label="Sayı"><b-form-input v-model.number="yeniUyruk.sayi" type="number" min="1"></b-form-input></b-form-group></div>
                  <div class="col-md-2 mb-3"><b-button variant="success" @click="addUyruk" class="w-100">Ekle</b-button></div>
                </div>
                <b-table striped hover :items="form.uyruklar" :fields="uyrukFields" responsive="sm" v-if="form.uyruklar && form.uyruklar.length > 0">
                    <template #cell(actions)="row">
                      <b-button v-if="!isDisabled" size="sm" variant="danger" @click="removeUyruk(row.index)">Sil</b-button>
                    </template>
                </b-table>
              </fieldset>
              <div v-if="!isDisabled" class="d-flex justify-content-between mt-3">
                <b-button variant="secondary" @click="prevTab">Geri</b-button>
                <b-button variant="primary" @click="nextTab">İleri</b-button>
              </div>
            </b-tab>

            <!-- TAB 3: VASITA BİLGİLERİ -->
            <b-tab title="3. Vasıta">
              <fieldset class="border p-3 rounded">
                <legend class="w-auto h5 text-primary">Vasıta Bilgileri</legend>
                 <div class="row">
                  <div class="col-md-4"><b-form-group label="Vasıta İsmi"><b-form-select v-model="form.vasitaIsmi" :options="vasitaIsmiOptions" required :disabled="isDisabled"><template #first><b-form-select-option :value="null" disabled>Vasıta ismi seçiniz</b-form-select-option></template></b-form-select></b-form-group></div>
                  <div class="col-md-4"><b-form-group label="Kullanılan Vasıta"><b-form-select v-model="form.kullanilanVasita" :options="kullanilanVasitaOptions" required :disabled="isDisabled"></b-form-select></b-form-group></div>
                  <div class="col-md-4"><b-form-group label="Kategori"><b-form-select v-model="form.kategori" :options="vasitaKategoriOptions" required :disabled="isDisabled"><template #first><b-form-select-option :value="null" disabled>Kategori seçiniz</b-form-select-option></template></b-form-select></b-form-group></div>
                </div>
                <div class="row">
                  <div class="col-md-4"><b-form-group label="Üretim Yeri"><b-form-select v-model="form.uretimYeri" :options="vasitaUretimYeriOptions" required :disabled="isDisabled"><template #first><b-form-select-option :value="null" disabled>Üretim yeri seçiniz</b-form-select-option></template></b-form-select></b-form-group></div>
                  <div class="col-md-4"><b-form-group label="Seri No"><b-form-select v-model="form.seriNo" :options="vasitaSeriNoOptions" required :disabled="isDisabled"><template #first><b-form-select-option :value="null" disabled>Seri no seçiniz</b-form-select-option></template></b-form-select></b-form-group></div>
                  <div class="col-md-4"><b-form-group label="Sağlam/Hasarlı"><b-form-select v-model="form.saglamHasarli" :options="[{text:'Sağlam', value:'Sağlam'}, {text:'Hasarlı', value:'Hasarlı'}]" required :disabled="isDisabled"></b-form-select></b-form-group></div>
                </div>
                <div class="row">
                  <div class="col-md-6"><b-form-group label="Benzin Var/Yok"><b-form-select v-model="form.benzinVarYok" :options="[{text:'Var', value:'Var'}, {text:'Yok', value:'Yok'}]" required :disabled="isDisabled"></b-form-select></b-form-group></div>
                  <div class="col-md-6"><b-form-group label="Hareketsiz"><b-form-select v-model="form.hareketsiz" :options="[{text:'Evet', value:'Evet'}, {text:'Hayır', value:'Hayır'}]" required :disabled="isDisabled"></b-form-select></b-form-group></div>
                </div>
              </fieldset>
              <div v-if="!isDisabled" class="d-flex justify-content-between mt-3">
                <b-button variant="secondary" @click="prevTab">Geri</b-button>
                <b-button variant="primary" @click="nextTab">İleri</b-button>
              </div>
            </b-tab>

            <!-- TAB 4: DETAYLI SAYILAR -->
            <b-tab title="4. Detaylı Sayılar">
              <fieldset class="border p-3 rounded">
                <legend class="w-auto h5 text-primary">Detaylı Sayılar (Sağ, Yaralı, Ölü vb.) *</legend>
                 <div v-if="!isDisabled" class="row align-items-end p-2 mb-2 bg-light border rounded">
                    <div class="col-md-4"><b-form-group label-class="font-weight-bold" label="Kategori"><b-form-select v-model="yeniKategori.kategori" :options="kategoriOptions"></b-form-select></b-form-group></div>
                    <div class="col-md-3"><b-form-group label-class="font-weight-bold" label="Durum (Cinsiyet)"><b-form-select v-model="yeniKategori.durum" :options="durumOptions"></b-form-select></b-form-group></div>
                    <div class="col-md-3"><b-form-group label-class="font-weight-bold" label="Sayı"><b-form-input v-model.number="yeniKategori.sayi" type="number" min="0"></b-form-input></b-form-group></div>
                    <div class="col-md-2 mb-3"><b-button variant="success" @click="addKategori" class="w-100">Ekle</b-button></div>
                </div>
                <!-- DEĞİŞİKLİK: Tablo verisi `form.kategoriler` oldu -->
                <b-table striped hover :items="form.kategoriler" :fields="kategoriFields" responsive="sm" v-if="form.kategoriler && form.kategoriler.length > 0">
                    <template #cell(actions)="row">
                      <b-button v-if="!isDisabled" size="sm" variant="danger" @click="removeKategori(row.index)">Sil</b-button>
                    </template>
                </b-table>
              </fieldset>
              <div v-if="!isDisabled" class="d-flex justify-content-between mt-3">
                <b-button variant="secondary" @click="prevTab">Geri</b-button>
                <b-button variant="primary" @click="nextTab">İleri</b-button>
              </div>
            </b-tab>

            <!-- TAB 5: DİĞER BİLGİLER VE KAYIT -->
            <b-tab title="5. Diğer Bilgiler ve Kayıt">
               <fieldset class="border p-3 rounded">
                <legend class="w-auto h5 text-primary">Diğer Bilgiler</legend>
                <div class="row">
                    <div class="col-md-6"><b-form-group label="Yakalanan Türk Sayısı" label-for="yakalanan-turk"><b-form-input v-model.number="form.yakalananTurk" type="number" min="0" required :disabled="isDisabled"></b-form-input></b-form-group></div>
                    <div class="col-md-6"><b-form-group label="Organizatör Sayısı" label-for="organizator"><b-form-input v-model.number="form.organizatorSayisi" type="number" min="0" required :disabled="isDisabled"></b-form-input></b-form-group></div>
                </div>
                <div class="row">
                    <div class="col-md-6"><b-form-group label="İhbar Tipi" label-for="ihbar-tipi"><b-form-select v-model="form.ihbarTipi" :options="ihbarTipiOptions" required :disabled="isDisabled"></b-form-select></b-form-group></div>
                    <div class="col-md-6"><b-form-group label="İhbar Kaynağı" label-for="ihbar-kaynagi" :required="form.ihbarTipi !== 'İhbarsız'"><b-form-select v-model="form.ihbarKaynagi" :options="ihbarKaynagiOptions" :disabled="form.ihbarTipi === 'İhbarsız' || isDisabled" :required="form.ihbarTipi !== 'İhbarsız'"></b-form-select></b-form-group></div>
                </div>
              </fieldset>
              <div v-if="!isDisabled" class="d-flex justify-content-between mt-4">
                <b-button variant="secondary" @click="prevTab">Geri</b-button>
                <div>
                  <b-button type="submit" variant="primary" :disabled="isSubmitting" class="mr-2">
                    <b-spinner v-if="isSubmitting" small class="mr-1"></b-spinner>
                    {{ isSubmitting ? 'Kaydediliyor...' : 'Kaydet' }}
                  </b-button>
                  <b-button type="button" variant="secondary" @click="resetForm">Temizle</b-button>
                </div>
              </div>
            </b-tab>
          </b-tabs>
        </form>
      </div>
    </div>

</div>
</template>

<script>
import axios from 'axios';

const getDefaultFormState = () => ({
  olayNo: '',
  olayTarihi: new Date().toISOString().split('T')[0],
  olaySaati: '00:00:00',
  bolge: null,
  deniz: null,
  il: null,
  ilce: null,
  yer: null,
  enlem: '',
  boylam: '',
  olayKategorisi: null,
  denizeAtma: null,
  yakalananTurk: 0,
  organizatorSayisi: 0,
  ihbarTipi: null,
  ihbarKaynagi: null,
  sgMi: true,
  uyruklar: [],
  vasitaIsmi: null,
  kullanilanVasita: null,
  kategori: null,
  uretimYeri: null,
  seriNo: null,
  benzinVarYok: 'Yok',
  saglamHasarli: 'Sağlam',
  hareketsiz: 'Hayır',
  kategoriler: [],
});


export default {
  name: 'GocmenOperasyonFormu',
  props: {
    initialData: {
      type: Object,
      default: () => ({})
    },
    isDisabled: {
      type: Boolean,
      default: false
    }
  },

  data() {
    return {
      tabIndex: 0,
      form: getDefaultFormState(), // Form state'ini fonksiyonla başlat
      yeniUyruk: { uyruk: null, sayi: 1 },
      yeniKategori: { kategori: null, durum: null, sayi: 0 },
      isSubmitting: false,
      modalTitle: '',
      modalMessage: '',
      modalVariant: 'success',
     
      ilOptions: [],
      ilceOptions: [],

      bolgeOptions: [ { value: 'AKDENİZ', text: 'AKDENİZ' }, { value: 'EGE', text: 'EGE' }, { value: 'KARADENİZ', text: 'KARADENİZ' }, { value: 'MARMARA', text: 'MARMARA' } ],
      denizOptions: [ { value: 'AKDENİZ', text: 'AKDENİZ' }, { value: 'EGE', text: 'EGE' }, { value: 'VAN GÖLÜ', text: 'VAN GÖLÜ' } ],
     
      uyrukOptions: [ { value: 'suriye', text: 'Suriye' }, { value: 'afganistan', text: 'Afganistan' }, { value: 'pakistan', text: 'Pakistan' }, { value: 'irak', text: 'Irak' }, { value: 'fas', text: 'Fas' }, { value: 'kamerun', text: 'Kamerun' }, { value: 'nijerya', text: 'Nijerya' }, { value: 'cad', text: 'Çad' }, { value: 'tunus', text: 'Tunus' }, { value: 'kenya', text: 'Kenya' }, { value: 'zambiya', text: 'Zambiya' }, { value: 'gana', text: 'Gana' }, { value: 'benin', text: 'Benin' }, { value: 'ozbekistan', text: 'Özbekistan' }, { value: 'uganda', text: 'Uganda' }, { value: 'etiyopya', text: 'Etiyopya' }, { value: 'burkino_faso', text: 'Burkina Faso' }, { value: 'kuveyt', text: 'Kuveyt' }, { value: 'nepal', text: 'Nepal' }, { value: 'togo', text: 'Togo' }, { value: 'haiti', text: 'Haiti' }, { value: 'liberya', text: 'Liberya' }, { value: 'komor_adalari', text: 'Komor Adaları' }, { value: 'urdun', text: 'Ürdün' }, { value: 'moritanya', text: 'Moritanya' }, { value: 'rusya', text: 'Rusya' }, { value: 'azerbaycan', text: 'Azerbaycan' }, { value: 'tanzanya', text: 'Tanzanya' }, { value: 'nijer', text: 'Nijer' }, { value: 'botsvana', text: 'Botsvana' } ],
      gecmeTesebbusOptions: [ { value: 'BULAMAÇ', text: 'BULAMAÇ' }, { value: 'DEDEAĞAÇ', text: 'DEDEAĞAÇ' }, { value: 'KKTC', text: 'KKTC' }, { value: 'LİMNİ', text: 'LİMNİ' }, { value: 'MEİS', text: 'MEİS' }, { value: 'MİDİLLİ', text: 'MİDİLLİ' }, { value: 'RODOS', text: 'RODOS' }, { value: 'SAKIZ', text: 'SAKIZ' }, { value: 'SÖMBEKİ', text: 'SÖMBEKİ' }, { value: 'SİSAM', text: 'SİSAM' }, { value: 'VAN', text: 'VAN' }, { value: 'İSTANKÖY', text: 'İSTANKÖY' }, { value: 'İTALYA', text: 'İTALYA' } ],
      vasitaIsmiOptions: [ { value: '2 CAN SALI', text: '2 CAN SALI' }, { value: '3 CAN SALI', text: '3 CAN SALI' }, { value: '4 CAN SALI', text: '4 CAN SALI' }, { value: 'ADA-34', text: 'ADA-34' }, { value: 'ADA/KARA ÜZERİNDEN', text: 'ADA/KARA ÜZERİNDEN' }, { value: 'ASTERİ', text: 'ASTERİ' }, { value: 'ASİL TÜRK', text: 'ASİL TÜRK' }, { value: 'ATABEY-35', text: 'ATABEY-35' }, { value: 'BODOTO 10', text: 'BODOTO 10' }, { value: 'BİR MASAL', text: 'BİR MASAL' }, { value: 'CAPTAIN ZÜLFÜ', text: 'CAPTAIN ZÜLFÜ' }, { value: 'ÇEPELİ', text: 'ÇEPELİ' }, { value: 'DEDEAĞAÇ', text: 'DEDEAĞAÇ' }, { value: 'DOLPHİ', text: 'DOLPHİ' }, { value: 'ELİF', text: 'ELİF' }, { value: 'EMİNE', text: 'EMİNE' }, { value: 'ERAY', text: 'ERAY' }, { value: 'FENİKS', text: 'FENİKS' }, { value: 'FRİXOS', text: 'FRİXOS' }, { value: 'GOOD MOON', text: 'GOOD MOON' }, { value: 'KARATAŞ 1', text: 'KARATAŞ 1' }, { value: 'KAZAN-1', text: 'KAZAN-1' }, { value: 'KIBRIS', text: 'KIBRIS' }, { value: 'KOTİL', text: 'KOTİL' }, { value: 'LİMON-3', text: 'LİMON-3' }, { value: 'MESUT REİS', text: 'MESUT REİS' }, { value: 'MUĞLA-1', text: 'MUĞLA-1' }, { value: 'MUSTAFA', text: 'MUSTAFA' }, { value: 'NURAY', text: 'NURAY' }, { value: 'ONUR', text: 'ONUR' }, { value: 'PARSS-1 VE ADA 34', text: 'PARSS-1 VE ADA 34' }, { value: 'RACİNG 1', text: 'RACİNG 1' }, { value: 'RAGNAR-47', text: 'RAGNAR-47' }, { value: 'RONYA', text: 'RONYA' }, { value: 'RÜZGAR REİS 01', text: 'RÜZGAR REİS 01' }, { value: 'S. AMAZON 35', text: 'S. AMAZON 35' }, { value: 'SKY 48', text: 'SKY 48' }, { value: 'SUVAARİ', text: 'SUVAARİ' }, { value: 'ŞAHİN-1', text: 'ŞAHİN-1' }, { value: 'T.C.G.S. 902', text: 'T.C.G.S. 902' }, { value: 'T.C.S.G. 311', text: 'T.C.S.G. 311' }, { value: 'TRİTON', text: 'TRİTON' }, { value: 'UMBRA', text: 'UMBRA' }, { value: 'VENESUELLA', text: 'VENESUELLA' }, { value: 'WHİTE GREAM', text: 'WHİTE GREAM' }, { value: 'YEDİTEPE-1', text: 'YEDİTEPE-1' }, { value: 'YUSUF REİS-3', text: 'YUSUF REİS-3' }, { value: 'YÜZEREK', text: 'YÜZEREK' }, { value: 'İSİMSİZ', text: 'İSİMSİZ' }, { value: 'ŞİŞME YATAK', text: 'ŞİŞME YATAK' } ],
      kullanilanVasitaOptions: [ { value: 'ADA/KARA ÜZERİNDEN', text: 'ADA/KARA ÜZERİNDEN' }, { value: 'AHŞAP TEKNE', text: 'AHŞAP TEKNE' }, { value: 'BALIK AVLAMA TEKNESİ', text: 'BALIK AVLAMA TEKNESİ' }, { value: 'BALIKÇI', text: 'BALIKÇI' }, { value: 'CAN SALI', text: 'CAN SALI' }, { value: 'DENİZ ÜZERİNDEN', text: 'DENİZ ÜZERİNDEN' }, { value: 'FERİBOT', text: 'FERİBOT' }, { value: 'FİBER KARİNALI LASTİK BOT', text: 'FİBER KARİNALI LASTİK BOT' }, { value: 'FİBER TEKNE', text: 'FİBER TEKNE' }, { value: 'GÖÇMEN BOT', text: 'GÖÇMEN BOT' }, { value: 'JANDARMA BOT', text: 'JANDARMA BOT' }, { value: 'LASTİK BOT', text: 'LASTİK BOT' }, { value: 'MASAÜSTÜ BOT', text: 'MASAÜSTÜ BOT' }, { value: 'MOTORLU ZODYAK', text: 'MOTORLU ZODYAK' }, { value: 'RIB BOT', text: 'RIB BOT' }, { value: 'SAHİL GÜVENLİK BOT', text: 'SAHİL GÜVENLİK BOT' }, { value: 'SAVUNMA BOT', text: 'SAVUNMA BOT' } ],
      vasitaKategoriOptions: [ { value: 'LASTİK BOT', text: 'LASTİK BOT' }, { value: 'FİBER KARİNALI LASTİK BOT', text: 'FİBER KARİNALI LASTİK BOT' }, { value: 'CAN SALI', text: 'CAN SALI' }, { value: 'SÜRAT TEKNESİ', text: 'SÜRAT TEKNESİ' } ],
      vasitaUretimYeriOptions: [ { value: 'SANCAK TEKNE (SANCAK 6,9)', text: 'SANCAK TEKNE (SANCAK 6,9)' } ],
      vasitaSeriNoOptions: [ { value: 'TR SEMA 4664J424MOTOR NO:3B624947 (150 HP)', text: 'TR SEMA 4664J424MOTOR NO:3B624947 (150 HP)' } ],
      uyrukFields: [ { key: 'uyruk', label: 'Uyruk' }, { key: 'sayi', label: 'Sayı' }, { key: 'actions', label: 'İşlemler', class: 'text-right' } ],
      kategoriFields: [ { key: 'kategori', label: 'Kategori' }, { key: 'durum', label: 'Durum (Cinsiyet)' }, { key: 'sayi', label: 'Sayı' }, { key: 'actions', label: 'İşlemler', class: 'text-right' } ],
      sgOptions: [ { text: 'SG', value: true }, { text: 'Müşterek', value: false } ],
      kategoriOptions: [ {value: 'Sağ', text: 'Sağ'}, {value: 'Yaralı', text: 'Yaralı'}, {value: 'Kayıp', text: 'Kayıp'}, {value: 'Ölü', text: 'Ölü'} ],
      durumOptions: [ {value: 'Erkek', text: 'Erkek'}, {value: 'Kadın', text: 'Kadın'}, {value: 'Erkek Çocuk', text: 'Erkek Çocuk'}, {value: 'Kız Çocuk', text: 'Kız Çocuk'} ],
      varYokOptions: [ { value: 'Var', text: 'Var' }, { value: 'Yok', text: 'Yok' } ],
      olayKategoriOptions: [ { value: 'Yakalama', text: 'Yakalama' }, { value: 'Geri İtme', text: 'Geri İtme' }, { value: 'Kurtarma', text: 'Kurtarma' } ],
      ihbarTipiOptions: [ { value: 'İhbarlı', text: 'İhbarlı' }, { value: 'İhbarsız', text: 'İhbarsız' } ],
      ihbarKaynagiOptions: [ { value: 'AAKKM', text: 'AAKKM' }, { value: 'ALO 112', text: 'ALO 112' }, { value: 'ÇEŞME KOLDESTİM', text: 'ÇEŞME KOLDESTİM' }, { value: 'ÇEŞMEKOLDES', text: 'ÇEŞMEKOLDES' }, { value: 'DİKİLİ KOLDES', text: 'DİKİLİ KOLDES' }, { value: 'JANDARMA', text: 'JANDARMA' }, { value: 'JRCC PİRE', text: 'JRCC PİRE' }, { value: 'KB-112', text: 'KB-112' }, { value: 'KB-113', text: 'KB-113' }, { value: 'KB-14', text: 'KB-14' }, { value: 'KB-20', text: 'KB-20' }, { value: 'KB-22', text: 'KB-22' }, { value: 'KB-22,KB-89', text: 'KB-22,KB-89' }, { value: 'KB-35', text: 'KB-35' }, { value: 'KB-39', text: 'KB-39' }, { value: 'KB-4510', text: 'KB-4510' }, { value: 'KB-73', text: 'KB-73' }, { value: 'KB-75', text: 'KB-75' }, { value: 'KB-88', text: 'KB-88' }, { value: 'KB-89', text: 'KB-89' }, { value: 'KÜÇÜKKUYU KOLDESTİM', text: 'KÜÇÜKKUYU KOLDESTİM' }, { value: 'MORAD', text: 'MORAD' }, { value: 'MORAD-11', text: 'MORAD-11' }, { value: 'MORAD-12', text: 'MORAD-12' }, { value: 'MORAD-20', text: 'MORAD-20' }, { value: 'NATO', text: 'NATO' }, { value: 'RODOS LİMAN BAŞKANLIĞI', text: 'RODOS LİMAN BAŞKANLIĞI' }, { value: 'SAGUVEGE İSTH.', text: 'SAGUVEGE İSTH.' }, { value: 'SAGÜVEGE İSTH.', text: 'SAGÜVEGE İSTH.' }, { value: 'SAGÜVGÜNEGE İSTH.', text: 'SAGÜVGÜNEGE İSTH.' }, { value: 'SG İHA', text: 'SG İHA' }, { value: 'SGRS ÇEŞME', text: 'SGRS ÇEŞME' }, { value: 'SGYS', text: 'SGYS' }, { value: 'TCSG-107', text: 'TCSG-107' }, { value: 'TCSG-24', text: 'TCSG-24' }, { value: 'TCSG-27', text: 'TCSG-27' }, { value: 'TCSG-30', text: 'TCSG-30' }, { value: 'TCSG-311', text: 'TCSG-311' }, { value: 'TCSG-6', text: 'TCSG-6' }, { value: 'TCSG-61', text: 'TCSG-61' }, { value: 'TCSG-84', text: 'TCSG-84' }, { value: 'TCSG-904', text: 'TCSG-904' }, { value: 'TCSG-907', text: 'TCSG-907' }, { value: 'TCSG-908, TCSG-903, TCSG-66', text: 'TCSG-908, TCSG-903, TCSG-66' } ],
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


  watch: {
    initialData: {
      handler(dataToLoad) {
        // Yeni veri geldiğinde formu doldurmak için bu metodu çağır
        if (dataToLoad && dataToLoad.olayId) {
          this.populateForm(dataToLoad);
        } else {
          // Eğer initialData boş gelirse, formu sıfırla (yeni kayıt modu için)
          this.resetForm();
        }
      },
      immediate: true, 
      deep: true       
    },
    'form.ihbarTipi'(newValue) {
      if (newValue === 'İhbarsız') {
        this.form.ihbarKaynagi = null;
      }
    }
  },

  created() {
    this.loadIller();
  },

  methods: {
    async populateForm(dataToLoad) {
      // 1. Gelen verinin derin bir kopyasını alarak formu doldur.
      // Bu, `uyruklar` ve `kategoriler` gibi dizilerin doğru şekilde yüklenmesini sağlar.
      const formData = JSON.parse(JSON.stringify(dataToLoad));
      
      // 2. Backend'den gelen 'vasitalar' dizisindeki ilk elemanı alıp formdaki ilgili alanlara ata.
      if (formData.vasitalar && formData.vasitalar.length > 0) {
        const vasitaData = formData.vasitalar[0];
        formData.vasitaIsmi = vasitaData.vasitaIsmi;
        formData.kullanilanVasita = vasitaData.kullanilanVasita;
        formData.kategori = vasitaData.kategori;
        formData.uretimYeri = vasitaData.uretimYeri;
        formData.seriNo = vasitaData.seriNo;
        formData.benzinVarYok = vasitaData.benzinVarYok;
        formData.saglamHasarli = vasitaData.saglamHasarli;
        formData.hareketsiz = vasitaData.hareketsiz;
      }

      // Backend'den gelen 'kategori' alanını, formdaki 'olayKategorisi' ile eşleştir
      formData.olayKategorisi = formData.kategori;

      // 3. İl/İlçe dropdown'larını ayarlama
      // Önce il options'larının yüklendiğinden emin ol
      if (this.ilOptions.length === 0) {
        await this.loadIller();
      }

      const selectedIlObj = this.ilOptions.find(opt => opt.text === formData.il);
      if (selectedIlObj) {
        formData.il = selectedIlObj.value; // Formdaki `il` değerini ID olarak ayarla
        await this.loadIlceOptions(formData.il); // İlçeleri yükle
        // İlçe değeri metin olarak geldiği için, onu da `value` olarak ayarla
        formData.ilce = dataToLoad.ilce;
      }
      
      // 4. Hazırlanan veriyi form'a ata
      this.form = formData;
    },

    handleTabActivation(newTabIndex, prevTabIndex, bvEvent) {
      // Salt okunur modda tab değiştirmeyi ENGELLEME, kullanıcının veriyi görmesine izin ver.
      if (this.isDisabled) {
        return; 
      }
      
      // Geri tuşuna basıldığında validasyon yapma
      if (newTabIndex < prevTabIndex) {
        return;
      }
      
      // İleri giderken önceki tüm tabları valide et
      for (let i = 0; i < newTabIndex; i++) {
        if (!this.validateTabByIndex(i)) {
          bvEvent.preventDefault(); 
          break; 
        }
      }
    },

    validateTabByIndex(index) {
      switch (index) {
        case 0: return this.validateAnaOlayTab();
        case 1: return this.validateUyrukTab();
        case 2: return this.validateVasitaTab();
        case 3: return this.validateDetayliSayiTab();
        default: return true;
      }
    },
    nextTab() {
      if (this.validateCurrentTab()) {
        this.tabIndex++;
      }
    },
    prevTab() {
      this.tabIndex--;
    },
    validateCurrentTab() {
      return this.validateTabByIndex(this.tabIndex);
    },
    validateAnaOlayTab() {
      const fields = [
        { key: 'olayNo', label: 'Olay No' }, { key: 'olayTarihi', label: 'Olay Tarihi' }, { key: 'olaySaati', label: 'Olay Saati' }, { key: 'bolge', label: 'Bölge' }, { key: 'deniz', label: 'Deniz' }, { key: 'yer', label: 'Geçme Teşebbüs Yeri' }, { key: 'il', label: 'İl' }, { key: 'ilce', label: 'İlçe' }, { key: 'enlem', label: 'Enlem' }, { key: 'boylam', label: 'Boylam' }, { key: 'olayKategorisi', label: 'Kategori' }, { key: 'denizeAtma', label: 'Denize Atma' }
      ];
      for (const field of fields) {
        const value = this.form[field.key];
        if (value === null || value === undefined || (typeof value === 'string' && value.trim() === '')) {
          this.showStatusModal('Eksik Bilgi', `Lütfen '1. Ana Olay' sekmesindeki '${field.label}' alanını doldurunuz.`, 'warning');
          return false;
        }
      }
      return true;
    },
    validateUyrukTab() {
      if (!this.form.uyruklar || this.form.uyruklar.length === 0) {
        this.showStatusModal('Eksik Bilgi', `Lütfen '2. Uyruklar' sekmesine en az bir uyruk bilgisi ekleyiniz.`, 'warning');
        return false;
      }
      return true;
    },
    validateVasitaTab() {
      const fields = [
        { key: 'vasitaIsmi', label: 'Vasıta İsmi' }, { key: 'kullanilanVasita', label: 'Kullanılan Vasıta' }, { key: 'kategori', label: 'Vasıta Kategori' }, { key: 'uretimYeri', label: 'Üretim Yeri' }, { key: 'seriNo', label: 'Seri No' }, { key: 'saglamHasarli', label: 'Sağlam/Hasarlı' }, { key: 'benzinVarYok', label: 'Benzin Durumu' }, { key: 'hareketsiz', label: 'Hareketsiz Durumu' }
      ];
      for (const field of fields) {
        const value = this.form[field.key];
        if (value === null || value === undefined) {
          this.showStatusModal('Eksik Bilgi', `Lütfen '3. Vasıta' sekmesindeki '${field.label}' alanını doldurunuz.`, 'warning');
          return false;
        }
      }
      return true;
    },
    validateDetayliSayiTab() {
       if (!this.form.kategoriler || this.form.kategoriler.length === 0) {
        this.showStatusModal('Eksik Bilgi', `Lütfen '4. Detaylı Sayılar' sekmesine en az bir detaylı sayı bilgisi ekleyiniz.`, 'warning');
        return false;
      }
      return true;
    },
    async loadIller() {
      try {
        const response = await axios.get('/api/GocmenOperasyon/iller');
        if (response.data.success) {
          this.ilOptions = response.data.data.map(il => ({
            value: il.ilId,
            text: il.ad
          }));
        } else {
          this.showStatusModal('Hata', 'İl listesi yüklenemedi.', 'danger');
        }
      } catch (error) {
        console.error('İl yükleme hatası:', error);
        this.showStatusModal('Sunucu Hatası', 'İl listesi yüklenirken bir sorun oluştu.', 'danger');
      }
    },
    async onIlChange() {
      this.form.ilce = null;
      this.ilceOptions = [];
      if (this.form.il) {
        await this.loadIlceOptions(this.form.il);
      }
    },

    async loadIlceOptions(ilId) {
      try {
        const response = await axios.get(`/api/GocmenOperasyon/ilceler/${ilId}`);
        if (response.data.success) {
          this.ilceOptions = response.data.data.map(ilce => ({
            value: ilce.ad,
            text: ilce.ad
          }));
        } else {
          this.ilceOptions = [];
        }
      } catch (error) {
        console.error('İlçe yükleme hatası:', error);
        this.ilceOptions = [];
        this.showStatusModal('Sunucu Hatası', 'İlçe listesi yüklenirken bir sorun oluştu.', 'danger');
      }
    },
    addUyruk() {
      if (!this.yeniUyruk.uyruk || !this.yeniUyruk.sayi || this.yeniUyruk.sayi <= 0) {
        this.showStatusModal('Uyarı', 'Lütfen uyruk seçip geçerli bir sayı giriniz.', 'warning');
        return;
      }
      this.form.uyruklar.push({ ...this.yeniUyruk });
      this.yeniUyruk.uyruk = null;
      this.yeniUyruk.sayi = 1;
    },
    removeUyruk(index) {
      this.form.uyruklar.splice(index, 1);
    },

    addKategori() {
      if (!this.yeniKategori.kategori || !this.yeniKategori.durum) {
        this.showStatusModal('Uyarı', 'Lütfen kategori ve durum (cinsiyet) seçiniz.', 'warning');
        return;
      }
      this.form.kategoriler.push({ ...this.yeniKategori });
      this.yeniKategori = { kategori: null, durum: null, sayi: 0 };
    },
    removeKategori(index) {
      this.form.kategoriler.splice(index, 1);
    },
    async submitForm() {
      this.isSubmitting = true;

      const digerBilgilerFields = [
        { key: 'ihbarTipi', label: 'İhbar Tipi' },
      ];
      for (const field of digerBilgilerFields) {
        if (!this.form[field.key]) {
          this.showStatusModal('Eksik Bilgi', `Lütfen '${field.label}' alanını doldurunuz.`, 'danger');
          this.isSubmitting = false;
          return;
        }
      }

      if (this.form.ihbarTipi === 'İhbarlı' && !this.form.ihbarKaynagi) {
          this.showStatusModal('Eksik Bilgi', `Lütfen 'İhbar Kaynağı' alanını doldurunuz.`, 'danger');
          this.isSubmitting = false;
          return;
      }
      
      if (!this.validateTabByIndex(0) || !this.validateTabByIndex(1) || !this.validateTabByIndex(2) || !this.validateTabByIndex(3)) {
          this.isSubmitting = false;
          return;
      }

      try {
        const olayTarihiDate = new Date(this.form.olayTarihi);
        
        const selectedIl = this.ilOptions.find(option => option.value === this.form.il);
       
        const payload = {
          ...this.form,
          il: selectedIl ? selectedIl.text : null,
          kategori: this.form.olayKategorisi,
         
          yil: olayTarihiDate.getFullYear(),
          ay: olayTarihiDate.getMonth() + 1,
          gun: olayTarihiDate.getDate(),
          toplamGocmen: this.form.uyruklar.reduce((total, item) => total + (Number(item.sayi) || 0), 0),
          yaraliToplam: this.form.kategoriler.filter(k => k.kategori === 'Yaralı').reduce((total, item) => total + (Number(item.sayi) || 0), 0),
          kayipToplam: this.form.kategoriler.filter(k => k.kategori === 'Kayıp').reduce((total, item) => total + (Number(item.sayi) || 0), 0),
          oluToplam: this.form.kategoriler.filter(k => k.kategori === 'Ölü').reduce((total, item) => total + (Number(item.sayi) || 0), 0),
          
          vasitalar: [{
            vasitaIsmi: this.form.vasitaIsmi,
            kullanilanVasita: this.form.kullanilanVasita,
            kategori: this.form.kategori,
            uretimYeri: this.form.uretimYeri,
            seriNo: this.form.seriNo,
            benzinVarYok: this.form.benzinVarYok,
            saglamHasarli: this.form.saglamHasarli,
            hareketsiz: this.form.hareketsiz,
          }]
        };
        
        delete payload.olayKategorisi;
        
        ['vasitaIsmi', 'kullanilanVasita', 'uretimYeri', 'seriNo', 'benzinVarYok', 'saglamHasarli', 'hareketsiz'].forEach(key => delete payload[key]);

        const response = await axios.post('/api/GocmenOperasyon', payload);

        if (response.data.success) {
          this.showStatusModal('Başarılı', 'Operasyon verisi başarıyla kaydedildi!', 'success');
          this.resetForm();
        } else {
          this.showStatusModal('Hata', response.data.message || 'Kayıt sırasında bir hata oluştu.', 'danger');
        }
      } catch (error) {
        console.error('Form submission error:', error);
        const errorMessage = error.response?.data?.message || 'Sunucu hatası oluştu. Lütfen tekrar deneyiniz.';
        this.showStatusModal('Sunucu Hatası', errorMessage, 'danger');
      } finally {
        this.isSubmitting = false;
      }
    },

    resetForm() {
      this.form = getDefaultFormState();
      this.yeniUyruk = { uyruk: null, sayi: 1 };
      this.yeniKategori = { kategori: null, durum: null, sayi: 0 };
      this.ilceOptions = [];
      this.tabIndex = 0;
    },
    showStatusModal(title, message, variant) {
      this.modalTitle = title;
      this.modalMessage = message;
      this.modalVariant = variant;
      this.$bvModal.show('status-modal');
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

::v-deep .nav-tabs .nav-link {
  color: #6c757d; 
  background-color: #f8f9fa; 
  border-bottom-width: 2px;
  font-weight: 500;
  transition: all 0.2s ease-in-out;
}

::v-deep .nav-tabs .nav-link.active {
  color: #fff; 
  background-color: #007bff; 
  border-color: #007bff #007bff #fff;
}

::v-deep .nav-tabs .nav-link:not(.active):hover {
  background-color: #e9ecef; 
  border-color: #e9ecef #e9ecef #dee2e6;
  color: #495057;
}
</style>