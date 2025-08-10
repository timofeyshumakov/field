<template>
  <div v-if="loading"></div>
  <div v-else>
    <!-- Блок по целевым аудиториям -->
    <v-card 
      class="mb-3 elevation-1"
      v-for="(audience, audienceName) in filteredGroupedByAudience"
      :key="audienceName"
    >
      <v-card-title class="secondary white--text">
        {{ audienceTitles.get(audienceName) || audienceName }}
      </v-card-title>
      <v-list rounded>
        <template v-for="(contact, index) in audience" :key="contact.ID">
          <v-list-item>
            <v-list-item-content>
              <v-list-item-title class="text-h6 font-weight-bold primary--text">
                <a class="name" target="_blank" :href="'https://' + domain + '/crm/contact/details/' + contact.ID + '/'">{{ contact.FULL_NAME }}</a>
                <v-btn v-if="enityId === 'CRM_DEAL'" class="delete" icon color="error" @click="deleteCard(contact.ID)" aria-label="Delete card"><v-icon>mdi-close</v-icon></v-btn>
              </v-list-item-title>
              <v-list-item-subtitle class="text-subtitle-1">
                Должность: {{ contact.POST }}
              </v-list-item-subtitle>
              <v-list-item-subtitle class="text-subtitle-1">
                Email: {{ contact.EMAIL && contact.EMAIL[0] ? contact.EMAIL[0].VALUE : '' }}
              </v-list-item-subtitle>
              <v-list-item-subtitle class="text-subtitle-1">
                Город: {{ cityTitles.get(String(contact.UF_CRM_1753083765)) || contact.UF_CRM_1753083765 || '' }}
              </v-list-item-subtitle>
              <v-list dense>
                <v-list-item 
                  v-for="(medication, m) in contact.medications" 
                  :key="m"
                  class="pl-0"
                >
                  <v-list-item-content>
                    <div class="d-flex align-center ga-2">
                      <span class="font-weight-medium medication">
                        <a target="_blank" :href="'https://' + domain + '/page/spravochniki/pgirfw/type/189/details/' + contact.UF_CRM_1750766630[m] + '/'">{{ medication}}</a>
                      </span>
                      <span class="font-weight-medium"> - </span>
                      <span class="text--darken-1">
                        {{ contact.directions[m].join(', ') }}
                      </span>
                    </div>
                  </v-list-item-content>
                </v-list-item>
              </v-list>
            </v-list-item-content>
          </v-list-item>
          <v-divider v-if="index < audience.length - 1" :key="`divider-audience-${audienceName}-${index}`"></v-divider>
        </template>
      </v-list>
      <v-divider></v-divider>
      <!-- Список препаратов для данной аудитории -->
      <v-list-item-subtitle class="item" v-if="filteredAudienceMedications[audienceName] && filteredAudienceMedications[audienceName].length > 0">
        <span>Препараты: </span>
        <a
          v-for="(medication, m) in filteredAudienceMedications[audienceName]" 
          :key="`med-${m}`"
          target="_blank" :href="'https://' + domain + '/page/spravochniki/pgirfw/type/189/details/' + medication.id + '/'"
        >{{ filteredAudienceMedications[audienceName].length !== m + 1 ? medication.title + ", " : medication.title }}</a>
      </v-list-item-subtitle>
      <!-- Список оборудования для данной аудитории -->
      <v-list-item-subtitle class="item" v-if="filteredAudienceEquipment[audienceName] && filteredAudienceEquipment[audienceName].length > 0">
        <span>Оборудование: </span>
        <a
          v-for="(equipment, e) in filteredAudienceEquipment[audienceName]" 
          :key="`equip-${e}`"
          class="pl-0"
          target="_blank" :href="'https://' + domain + '/crm/type/1104/details/' + equipment.id + '/'"
        >{{ filteredAudienceEquipment[audienceName].length !== e + 1 ? equipment.title + ", " : equipment.title}}</a>
      </v-list-item-subtitle>
    </v-card>

    <!-- Блок Другие (контакты без целевой аудитории) -->
    <v-card 
      class="mb-3 elevation-1"
      v-if="contactsWithoutAudience.length > 0"
    >
      <v-card-title class="secondary white--text">
        Другие
      </v-card-title>
      <v-list rounded>
        <template v-for="(contact, index) in contactsWithoutAudience" :key="contact.ID">
          <v-list-item>
            <v-list-item-content>
              <v-list-item-title class="text-h6 font-weight-bold primary--text">
                <a class="name" target="_blank" :href="'https://' + domain + '/crm/contact/details/' + contact.ID + '/'">{{ contact.FULL_NAME }}</a>
                <v-btn v-if="enityId === 'CRM_DEAL'" class="delete" icon color="error" @click="deleteCard(contact.ID)" aria-label="Delete card"><v-icon>mdi-close</v-icon></v-btn>
              </v-list-item-title>
              <v-list-item-subtitle class="text-subtitle-1">
                Должность: {{ contact.POST }}
              </v-list-item-subtitle>
              <v-list-item-subtitle class="text-subtitle-1">
                Email: {{ contact.EMAIL && contact.EMAIL[0] ? contact.EMAIL[0].VALUE : '' }}
              </v-list-item-subtitle>
              <v-list-item-subtitle class="text-subtitle-1">
                Город: {{ cityTitles.get(String(contact.UF_CRM_1753083765)) || contact.UF_CRM_1753083765 || '' }}
              </v-list-item-subtitle>
              <div>
                <div
                  v-for="(medication, m) in contact.medications" 
                  :key="m"
                  class="pl-0"
                >
                  <v-list-item-content>
                    <div class="d-flex align-center ga-2">
                      <span class="font-weight-medium medication">
                        <a target="_blank" :href="'https://' + domain + '/page/baza_preparatov/preparaty/type/189/details/' + contact.UF_CRM_1750766630[m] + '/'">{{ medication}}</a>
                      </span>
                      <span class="font-weight-medium"> - </span>
                      <span class="text--darken-1">
                        {{ contact.directions[m].join(', ') }}
                      </span>
                    </div>
                  </v-list-item-content>
                </div>
              </div>
            </v-list-item-content>
          </v-list-item>
          <v-divider v-if="index < contactsWithoutAudience.length - 1" :key="`divider-other-${index}`"></v-divider>
        </template>
      </v-list>
    </v-card>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue';
import { callApi } from "../functions/callApi";

export default {
  setup() {
    const loading = ref(false);
    const error = ref(null);
    const contacts = ref([]);
    const contactsIds = ref([]);
    const domain = ref('');
    const audienceTitles = ref(new Map());
    const cityTitles = ref(new Map());
    const products = ref([]);
    const equipment = ref([]);
    const companyId = ref(0);
    const dealId = ref(0);
    const enityId = ref("");
    const dealAudience = ref([]);

    const fetchCompanyData = async () => {
      domain.value = BX24.getDomain();
      enityId.value = BX24.placement.info().options.ENTITY_ID;
      let contacts = "";

      try {
        if (enityId.value === "CRM_DEAL") {
          const deal = await callApi("crm.deal.list", { ID: BX24.placement.info().options.ENTITY_VALUE_ID }, ["COMPANY_ID", "UF_CRM_1754290331", "UF_CRM_1753365812"]);
          dealId.value = deal[0].ID;
          companyId.value = deal[0].COMPANY_ID;
          contacts = deal[0].UF_CRM_1754290331;
          dealAudience.value = deal[0].UF_CRM_1753365812 || [];
        } else if(enityId.value === "CRM_COMPANY") {
          companyId.value = BX24.placement.info().options.ENTITY_VALUE_ID;
          contacts = await new Promise((resolve, reject) => {
            BX24.callMethod("crm.company.contact.items.get", { id: companyId.value },
              (response) => {
                if (response.error()) {
                  reject(response.error());
                } else {
                  resolve(response.data().map((item) => item.CONTACT_ID));
                }
              }
            );
          });
        }

        const company = await new Promise((resolve, reject) => {
          BX24.callMethod("crm.company.get", { id: companyId.value },
            (response) => {
              if (response.error()) {
                reject(response.error());
              } else {
                resolve(response.data());
              }
            }
          );
        });
        return [contacts, company.UF_CRM_1745407296, company.UF_CRM_1753257731];
      } catch (err) {
        console.error('Ошибка:', err);
        error.value = err.message;
        throw err;
      }
    };
    
    const deleteCard = (id) => {
      contacts.value = contacts.value.filter(item => item.ID != id);
      BX24.callMethod("crm.deal.update", {id: dealId.value, fields: { UF_CRM_1754290331: contacts.value}},
      () => {});
    }

    const fetchData = async () => {
      try {
        loading.value = true;
        const [users, productsToFetch, equipmentToFetch] = await fetchCompanyData();

        const userData = await callApi("crm.contact.list", { ID: users }, ["UF_CRM_1750766630", "NAME", "LAST_NAME", "SECOND_NAME", "POST", "TYPE_ID", "EMAIL", "UF_CRM_1753364801", "UF_CRM_1753083765"]);
        
        products.value = await callApi("crm.item.list", {id: productsToFetch}, ["id", "title", "ufCrm26_1753365041"], 189);
        equipment.value = await callApi("crm.item.list", {id: equipmentToFetch}, ["id", "title", "ufCrm62_1753365319"], 1104);
        
        const productsFields = await new Promise((resolve, reject) => {
          BX24.callMethod('lists.element.get', {
            IBLOCK_TYPE_ID: 'lists',
            IBLOCK_ID: '216',
            SELECT: ['ID', 'NAME']
          }, (result) => {
            if (result.error()) {
              reject(new Error(`Ошибка: ${result.error().message}`));
            } else {
              resolve(result.data());
            }
          });
        });
        
        const cities = await callApi("crm.item.list", {}, ["id", "title"], 1094, 0, 0);
        
        audienceTitles.value = new Map(productsFields.map(item => [item.ID, item.NAME]));
        cityTitles.value = new Map(cities.map(item => [String(item.id), item.title]));

        userData.forEach(contact => {
          contact.FULL_NAME = `${contact.LAST_NAME ? contact.LAST_NAME : ""} ${contact.NAME ? contact.NAME : ""} ${contact.SECOND_NAME ? contact.SECOND_NAME : ""}`;
          if (contact.UF_CRM_1750766630 && contact.UF_CRM_1750766630.length > 0) {
            const productIds = contact.UF_CRM_1750766630.map(id => Number(id));

            const medications = productIds
              .map(id => products.value.find(product => product.id == Number(id)))
              .map(product => product && product.title)
              .filter(name => name !== undefined);

            const directions = productIds
              .map(id => products.value.find(product => product.id === Number(id)))
              .map(product => {
                if (!product) return [];
                let values = Array.isArray(product.ufCrm26_1753365041) ?
                  product.ufCrm26_1753365041 :
                  [product.ufCrm26_1753365041];
                return values.map(value => audienceTitles.value.get(String(value)) || String(value)).filter(d => d !== undefined);
              });

            contact.medications = medications;
            contact.directions = directions;
          }
        });

        contacts.value = userData;
        contactsIds.value = userData.map((item) => item.ID);
      } catch (err) {
        error.value = err.message || 'Ошибка загрузки данных';
        console.error('Ошибка:', err);
      } finally {
        loading.value = false;
      }
    };

    const filteredGroupedByAudience = computed(() => {
      const groups = new Map();
      contacts.value
        .filter(contact => contact.UF_CRM_1753364801 && contact.UF_CRM_1753364801.length > 0)
        .forEach(contact => {
          contact.UF_CRM_1753364801.forEach(audienceId => {
            const audienceTitle = audienceTitles.value.get(String(audienceId)) || String(audienceId);
            if (enityId.value !== 'CRM_DEAL' || dealAudience.value.includes(audienceId)) {
              if (!groups.has(audienceTitle)) {
                groups.set(audienceTitle, []);
              }
              if (!groups.get(audienceTitle).some(c => c.ID === contact.ID)) {
                groups.get(audienceTitle).push(contact);
              }
            }
          });
        });
      return Object.fromEntries(groups);
    });

    const filteredAudienceMedications = computed(() => {
      const medsByAudience = {};
      products.value.forEach(product => {
        let audiences = Array.isArray(product.ufCrm26_1753365041) ? 
          product.ufCrm26_1753365041 : 
          [product.ufCrm26_1753365041];
        audiences.forEach(audienceId => {
          const audienceTitle = audienceTitles.value.get(String(audienceId)) || String(audienceId);
          if (enityId.value !== 'CRM_DEAL' || dealAudience.value.includes(audienceId)) {
            if (!medsByAudience[audienceTitle]) {
              medsByAudience[audienceTitle] = [];
            }
            if (!medsByAudience[audienceTitle].some(med => med.id === product.id)) {
              medsByAudience[audienceTitle].push({ id: product.id, title: product.title });
            }
          }
        });
      });
      return medsByAudience;
    });

    const filteredAudienceEquipment = computed(() => {
      const equipByAudience = {};
      equipment.value.forEach(equip => {
        let audiences = Array.isArray(equip.ufCrm62_1753365319) ? 
          equip.ufCrm62_1753365319 : 
          [equip.ufCrm62_1753365319];
        audiences.forEach(audienceId => {
          const audienceTitle = audienceTitles.value.get(String(audienceId)) || String(audienceId);
          if (enityId.value !== 'CRM_DEAL' || dealAudience.value.includes(audienceId)) {
            if (!equipByAudience[audienceTitle]) {
              equipByAudience[audienceTitle] = [];
            }
            if (!equipByAudience[audienceTitle].some(e => e.id === equip.id)) {
              equipByAudience[audienceTitle].push({ id: equip.id, title: equip.title });
            }
          }
        });
      });
      return equipByAudience;
    });

    const contactsWithoutAudience = computed(() => {
      return contacts.value.filter(contact => !contact.UF_CRM_1753364801 || contact.UF_CRM_1753364801.length === 0);
    });

    onMounted(() => {
      fetchData();
    });

    return {
      loading,
      error,
      contacts,
      domain,
      filteredGroupedByAudience,
      contactsWithoutAudience,
      audienceTitles,
      cityTitles,
      filteredAudienceMedications,
      filteredAudienceEquipment,
      deleteCard,
      enityId
    };
  }
}
</script>