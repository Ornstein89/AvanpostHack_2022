<template>
  <v-container>
    <v-row>
      <v-col>
        <v-select
          label="Выберите действие"
          :items="analyzers"
          v-model="activeAnalyzer"
        />
        <v-form v-if="activeAnalyzer == '/api/recognize/'">
          <v-file-input label="Файл отпечатка пальца" v-model="firstImage" />
        </v-form>
        <v-form v-else>
          <v-row
            ><v-col>
              <v-file-input
                label="Первый файл отпечатка пальца"
                v-model="firstImage" /></v-col
            ><v-col v-if="firstImage"
              ><v-img width="200" :src="firstImageURL" /></v-col
          ></v-row>
          <v-row
            ><v-col
              ><v-file-input
                label="Второй файл отпечатка пальца"
                v-model="secondImage" /></v-col
            ><v-col v-if="secondImage"
              ><v-img width="200" :src="secondImageURL" /></v-col
          ></v-row>
          <v-checkbox
            label="Вернуть дополнительную информацию"
            v-model="verbose"
          />
        </v-form>
        <v-btn :loading="loading" @click="startAction">Запустить</v-btn>
      </v-col>
    </v-row>
    <v-row v-if="result">
      <v-col>
        <v-card
          ><v-card-title>Результат выполнения</v-card-title
          ><v-card-text
            ><p>Схожесть: {{ result.similarity }}</p>
            <p>
              Время выполнения на сервере: {{ result.executionTime }} сек.
            </p></v-card-text
          ></v-card
        >
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import http from "@/http";
export default {
  data() {
    return {
      analyzers: [
        {
          text: "Сравнение алгоритмом SIFT",
          value: "/api/sift/",
        },
        {
          text: "Сравнение нейросетью",
          value: "/api/tf/",
        },
        {
          text: "Информация о пользователе по отпечатку",
          value: "/api/recognize/",
        },
      ],
      activeAnalyzer: "/api/tf/",
      firstImage: null,
      secondImage: null,
      verbose: false,
      loading: false,
      result: null,
    };
  },
  watch: {
    activeAnalyzer() {
      this.result = null;
      this.firstImage = null;
      this.secondImage = null;
      this.verbose = false;
    },
  },
  computed: {
    firstImageURL() {
      if (this.firstImage) return URL.createObjectURL(this.firstImage);
      else return null;
    },
    secondImageURL() {
      if (this.firstImage) return URL.createObjectURL(this.secondImage);
      else return null;
    },
  },
  methods: {
    async getBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => {
          let encoded = reader.result.toString().replace(/^data:(.*,)?/, "");
          if (encoded.length % 4 > 0) {
            encoded += "=".repeat(4 - (encoded.length % 4));
          }
          resolve(encoded);
        };
        reader.onerror = (error) => reject(error);
      });
    },
    async startAction() {
      if (!this.firstImage) return;
      this.loading = true;
      this.result = null;
      const data = {
        firstImage: await this.getBase64(this.firstImage),
        verbose: this.verbose,
      };
      if (this.secondImage) {
        data.secondImage = await this.getBase64(this.secondImage);
      }
      const response = await http.createItem(this.activeAnalyzer, {
        data,
        showSnackbar: true,
      });
      if (response.status == 200) {
        this.result = response.data;
      }
      this.loading = false;
    },
  },
};
</script>

<style>
</style>