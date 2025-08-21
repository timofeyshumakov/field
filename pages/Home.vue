<template>
    <v-container>
      <v-card>
        <v-card-title>
          <button @click="showDeals()">Показать сделки</button>
          <h3>Data Table</h3>
        </v-card-title>
        <v-data-table
          :headers="headers"
          :items="deals"
          class="elevation-1"
        >
        </v-data-table>
      </v-card>
    </v-container>
  </template>
  
<script>
  import { ref, onMounted } from 'vue';
  
  export default {
    setup() {
      const deals = ref([]);
      const headers = [
        { text: 'ID', value: 'id' },
        { text: 'Opportunity', value: 'opportunity' },
      ];
      },
        methods: {
          showDeals() {
            BX24.init(() => {
              BX24.callMethod('userfieldtype.update', {
                USER_TYPE_ID: 'assigned',
                HANDLER: 'https://cdn-ru.bitrix24.ru/b5113579/app_local/3d9e59bcff7b35043d245a99b54bcb49/index.html',
                TITLE: 'Ответственный подробно',
              })
            });
        }
    },
  };
</script>
  
<style>
</style>