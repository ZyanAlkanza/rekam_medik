<template>
  <q-page class="q-pa-md">
    <q-card>
      <q-card-section class="logo-section">
        <q-img src="logo.svg" class="logo" />
      </q-card-section>
      <q-card-section>
        <q-btn rounded class="q-mr-sm" label="TAMBAH PASIEN" color="primary" @click="ModalTambahPasien = true" />
        <q-btn rounded label="BUAT KUNJUNGAN" color="green" @click="ModalKunjungan = true" />
      </q-card-section>
      <q-card-section>
        <q-table :rows="patients" :columns="columns" row-key="id">
          <template v-slot:body-cell-no="props">
            <q-td :props="props">
              {{ props.rowIndex + 1 }}
            </q-td>
          </template>
        </q-table>
      </q-card-section>

      <!-- Modal Form Tambah Pasien -->
      <q-dialog v-model="ModalTambahPasien">
        <q-card class="modal">
          <q-card-section class="top-modal row">
            <div class="title text-h6">TAMBAH PASIEN</div>
            <q-btn icon="close" flat round dense v-close-popup />
          </q-card-section>

          <q-form @submit.prevent="submitForm" class="form-spacing">
            <q-input disable standout label="No Rekam Medik" />
            <div class="row justify-between">
              <q-input class="col-6" standout v-model="newPatient.nama_pasien" label="Nama Pasien" autofocus
                :rules="[val => !/\d/.test(val) || 'Name must not contain number', val => !!val || 'Field is required', val => val.length >= 3 || 'Name must contain at least 3 characters']" />
              <q-input class="col-6" standout v-model="newPatient.nik" label="NIK" mask="################"
                :rules="[val => val.length === 16 || 'NIK must be 16 digits']" />
            </div>
            <q-input standout v-model="newPatient.alamat" label="Alamat" type="textarea" rows="4"
              class="textarea-address" :rules="[val => !!val || 'Field is required']" />

            <div class="row justify-center q-mt-lg">
              <q-btn class="button" label="Simpan" color="primary" type="submit" />
            </div>
          </q-form>
        </q-card>
      </q-dialog>

      <!-- Modal Form Kunjungan -->
      <q-dialog v-model="ModalKunjungan">
        <q-card class="modal">
          <q-card-section class="top-modal row">
            <div class="title text-h6">BUAT KUNJUNGAN</div>
            <q-btn icon="close" flat round dense v-close-popup />
          </q-card-section>

          <q-form @submit.prevent="submitFormKunjungan" class="form-spacing">
            <q-select standout v-model="selectedNoRekamMedik" use-input input-debounce="0" label="No Rekam Medik"
              :options="filteredOptions" @filter="filterFn" autofocus>
              <template v-slot:no-option>
                <q-item>
                  <q-item-section class="text-grey">
                    No results
                  </q-item-section>
                </q-item>
              </template>
            </q-select>

            <div class="row justify-between">
              <q-input disable class="col-6" standout v-model="newPatient.nama_pasien" label="Nama Pasien"
                :rules="[val => !/\d/.test(val) || 'Name must not contain number', val => !!val || 'Field is required', val => val.length >= 3 || 'Name must contain at least 3 characters']" />
              <q-input disable class="col-6" standout v-model="newPatient.nik" label="NIK" mask="################"
                :rules="[val => val.length === 16 || 'NIK must be 16 digits']" />
            </div>
            <q-input disable standout v-model="newPatient.alamat" label="Alamat" type="textarea" rows="4"
              class="textarea-address" :rules="[val => !!val || 'Field is required']" />

            <div class="row justify-center q-mt-lg">
              <q-btn class="button" label="Buat Kunjungan" color="primary" type="submit" />
            </div>
          </q-form>
        </q-card>
      </q-dialog>

    </q-card>
  </q-page>
</template>

<script>
import { Notify } from 'quasar';
import { api } from 'src/boot/axios';
import { defineComponent, watch } from 'vue';

export default defineComponent({
  name: 'DashboardPage',

  data() {
    return {
      columns: [
        { name: 'no', label: 'No', align: 'center' },
        { name: 'no_rekam_medik', label: 'No Rekam Medik', align: 'center', field: row => row.no_rekam_medik },
        { name: 'nama_pasien', label: 'Nama Pasien', align: 'left', field: row => row.nama_pasien },
        { name: 'nik', label: 'NIK', align: 'center', field: row => row.nik },
        { name: 'alamat', label: 'Alamat', align: 'left', field: row => row.alamat },
        { name: 'jumlah_kunjungan', label: 'Jumlah Kunjungan', align: 'center', field: row => row.visit ? row.visit.jumlah_kunjungan : '0' }
      ],
      patients: [],
      ModalTambahPasien: false,
      ModalKunjungan: false,
      newPatient: {
        nama_pasien: '',
        nik: '',
        alamat: '',
      },
      selectedNoRekamMedik: null,
      options: [],
      filteredOptions: []
    };
  },
  methods: {
    fetchPatients() {
      api.get('http://127.0.0.1:8000/api/index')
        .then(response => {
          this.patients = response.data.data;
          this.options = response.data.data.map(patient => ({
            label: patient.no_rekam_medik,
            value: patient.no_rekam_medik
          }));
          this.filteredOptions = this.options;
        })
        .catch(error => {
          console.error('Error fetching patients:', error);
        });
    },
    submitForm() {
      api.post('http://127.0.0.1:8000/api/add_patient', this.newPatient)
        .then(response => {
          this.fetchPatients();
          Notify.create({
            type: 'positive',
            message: 'Data added successfully!',
            position: 'top-right',
            timeout: 2500
          });
          this.newPatient = {
            nama_pasien: '',
            nik: '',
            alamat: '',
          };
          this.ModalTambahPasien = false;
        })
        .catch(error => {
          Notify.create({
            type: 'negative',
            message: 'Failed! ' + error.response.data.message,
            position: 'top-right',
            timeout: 2500
          });
          console.error('Error adding patient:', error);
        });
    },
    filterFn(val, update) {
      if (val === '') {
        update(() => {
          this.filteredOptions = this.options;
        });
        return;
      }

      const needle = val.toLowerCase();
      update(() => {
        this.filteredOptions = this.options.filter(v => v.label.toLowerCase().indexOf(needle) > -1);
      });
    },
    submitFormKunjungan() {
      api.post('http://127.0.0.1:8000/api/make_visit', {
        no_rekam_medik: this.selectedNoRekamMedik.value
      })
        .then(response => {
          this.fetchPatients();
          Notify.create({
            type: 'positive',
            message: 'Successfully!',
            position: 'top-right',
            timeout: 2500
          });
          this.ModalKunjungan = false;
          setTimeout(() => {
            this.selectedNoRekamMedik = null;
            this.newPatient = {
              nama_pasien: '',
              nik: '',
              alamat: '',
            };
          }, 500);
        })
        .catch(error => {
          Notify.create({
            type: 'negative',
            message: 'Failed! ' + error.response.data.message,
            position: 'top-right',
            timeout: 2500
          });
          console.log(error);
        })
    },
  },
  watch: {
    selectedNoRekamMedik(newVal) {
      const selectedPatient = this.patients.find(patient => patient.no_rekam_medik === newVal.value);
      console.log(newVal.value);


      if (selectedPatient) {
        this.newPatient = {
          nama_pasien: selectedPatient.nama_pasien,
          nik: selectedPatient.nik,
          alamat: selectedPatient.alamat,
        };
      } else {
        console.log('Patient not found for No Rekam Medik:', newVal);
      }
    }
  },
  mounted() {
    this.fetchPatients();
  },
});
</script>

<style scoped>
.q-page {
  background-color: #0561ce;
  padding: 80px;
}

.q-card {
  border-radius: 24px;
}

.q-table {
  min-width: 600px;
}

.logo-section {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
}

.logo {
  max-width: 200px;
  max-height: 100px;
  object-fit: cover;
}

.modal {
  width: 500px;
  padding: 24px;
}

.top-modal {
  display: flex;
}

.title {
  text-align: center;
  width: 90%;
}

.form-spacing>* {
  margin-bottom: 8px;
}

.button {
  width: 200px;
}
</style>
