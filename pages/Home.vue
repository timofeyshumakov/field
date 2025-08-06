<template>
  <div v-if="loading"></div>
  <div v-else>
    <!-- Блок по целевым аудиториям -->
    <v-card 
      class="mb-3 elevation-1"
      v-for="(audience, audienceName) in groupedByAudience"
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
                <v-btn class="delete" icon color="error" @click="deleteCard(contact.ID)" aria-label="Delete card"><v-icon>mdi-close</v-icon></v-btn>
              </v-list-item-title>
              <v-list-item-subtitle class="text-subtitle-1">
                {{ contact.POST }}
              </v-list-item-subtitle>
              <v-list-item-subtitle class="text-subtitle-1">
                {{ contact.EMAIL && contact.EMAIL[0] ? contact.EMAIL[0].VALUE : '' }}
              </v-list-item-subtitle>
              <v-list-item-subtitle class="text-subtitle-1">
                {{ cityTitles.get(String(contact.UF_CRM_1753083765)) || contact.UF_CRM_1753083765 || '' }}
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
                        <a target="_blank" :href="'https://' + domain + '/page/baza_preparatov/preparaty/type/189/details/' + contact.UF_CRM_1750766630[m] + '/'">{{ medication}}</a>
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
      <v-list-item-subtitle class="item" v-if="audienceMedications[audienceName] && audienceMedications[audienceName].length > 0">
        <span>Препараты: </span>
        <a
          v-for="(medication, m) in audienceMedications[audienceName]" 
          :key="`med-${m}`"
          target="_blank" :href="'https://' + domain + '/page/baza_preparatov/preparaty/type/189/details/' + medication.id + '/'"
        >{{ audienceMedications[audienceName].length !== m + 1 ? medication.title + ", " : medication.title }}</a>
      </v-list-item-subtitle>
      <!-- Список оборудования для данной аудитории -->
      <v-list-item-subtitle class="item" v-if="audienceEquipment[audienceName] && audienceEquipment[audienceName].length > 0">
        <span>Оборудование: </span>
        <a
          v-for="(equipment, e) in audienceEquipment[audienceName]" 
          :key="`equip-${e}`"
          class="pl-0"
          target="_blank" :href="'https://' + domain + '/crm/type/1104/details/' + equipment.id + '/'"
        >{{ audienceEquipment[audienceName].length !== e + 1 ? equipment.title + ", " : equipment.title}}</a>
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
                <v-btn class="delete" icon color="error" @click="deleteCard(contact.ID)" aria-label="Delete card"><v-icon>mdi-close</v-icon></v-btn>
              </v-list-item-title>
              <v-list-item-subtitle class="text-subtitle-1">
                {{ contact.POST }}
              </v-list-item-subtitle>
              <v-list-item-subtitle class="text-subtitle-1">
                {{ contact.EMAIL && contact.EMAIL[0] ? contact.EMAIL[0].VALUE : '' }}
              </v-list-item-subtitle>
              <v-list-item-subtitle class="text-subtitle-1">
                {{ cityTitles.get(String(contact.UF_CRM_1753083765)) || contact.UF_CRM_1753083765 || '' }}
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

    const fetchCompanyData = async () => {
      domain.value = BX24.getDomain();
      try {
        if (BX24.placement.info().options.ENTITY_ID === "CRM_DEAL") {
          const deal = await callApi("crm.deal.list", { ID: BX24.placement.info().options.ENTITY_VALUE_ID }, ["UF_CRM_1754290331"]);
          companyId.value = deal[0].UF_CRM_1754290331;
        } else if(BX24.placement.info().options.ENTITY_ID === "CRM_COMPANY") {
          companyId.value = BX24.placement.info().options.ENTITY_VALUE_ID;
        }
          const contacts = await new Promise((resolve, reject) => {
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
            BX24.callMethod("crm.contact.company.delete", {id: id, fields: { COMPANY_ID: companyId.value}},
            (result) => {
                if(result.error()){
                  console.error(result.error())
                }else{
                  contacts.value = contacts.value.filter(item => item.ID != id);
                }
            });
    }

    const fetchData = async () => {
      try {
        loading.value = true;
        // Получаем данные компании
        const [users, productsToFetch, equipmentToFetch] = await fetchCompanyData();

        // Получаем данные пользователя
        const userData = await callApi("crm.contact.list", { ID: users }, ["UF_CRM_1750766630", "NAME", "LAST_NAME", "SECOND_NAME", "POST", "TYPE_ID", "EMAIL", "UF_CRM_1753109871", "UF_CRM_1753083765"]);
        
        // Запрашиваем продукты и их поля
        products.value = await callApi("crm.item.list", {id: productsToFetch}, ["id", "title", "ufCrm26_1753110034"], 189);
        equipment.value = await callApi("crm.item.list", {id: equipmentToFetch}, ["id", "title", "ufCrm62_1753257668"], 1104);
        
        // Получаем целевые аудитории и их названия
        const productsFields = await callApi("crm.item.list", {}, ["id", "title"], 1098, 0, 0);
        
        // Получаем города и их названия (предполагаем, что города хранятся в entityId 1099)
        const cities = await callApi("crm.item.list", {}, ["id", "title"], 1094, 0, 0);
        
        // Создаем Map для сопоставления ID аудитории с названием
        audienceTitles.value = new Map(productsFields.map(item => [String(item.id), item.title]));
        
        // Создаем Map для сопоставления ID города с названием
        cityTitles.value = new Map(cities.map(item => [String(item.id), item.title]));

        // Переводим поля пользователей, заменяя ID аудиторий на названия
        userData.forEach(contact => {
          contact.FULL_NAME = `${contact.LAST_NAME ? contact.LAST_NAME : ""} ${contact.NAME ? contact.NAME : ""} ${contact.SECOND_NAME ? contact.SECOND_NAME : ""}`;
          if (contact.UF_CRM_1750766630 && contact.UF_CRM_1750766630.length > 0) {
            // Собираем продукты
            const productIds = contact.UF_CRM_1750766630.map(id => Number(id));

            // Список препаратов (продуктов)
            const medications = productIds
              .map(id => products.value.find(product => product.id === Number(id)))
              .map(product => product && product.title)
              .filter(name => name !== undefined);

            // Структура направлений (для каждого препарата может быть несколько направлений)
            const directions = productIds
              .map(id => products.value.find(product => product.id === Number(id)))
              .map(product => {
                if (!product) return [];
                let values = Array.isArray(product.ufCrm26_1753110034) ?
                  product.ufCrm26_1753110034 :
                  [product.ufCrm26_1753110034];
                return values.map(value => audienceTitles.value.get(String(value)) || String(value)).filter(d => d !== undefined);
              });

            // Присваиваем новые свойства контакта
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

    // Группировка контактов по целевой аудитории
    const groupedByAudience = computed(() => {
      const groups = new Map();
      contacts.value
        .filter(contact => contact.UF_CRM_1753109871 && contact.UF_CRM_1753109871.length > 0)
        .forEach(contact => {
          contact.UF_CRM_1753109871.forEach(audienceId => {
            const audienceTitle = audienceTitles.value.get(String(audienceId)) || String(audienceId);
            if (!groups.has(audienceTitle)) {
              groups.set(audienceTitle, []);
            }
            if (!groups.get(audienceTitle).some(c => c.ID === contact.ID)) {
              groups.get(audienceTitle).push(contact);
            }
          });
        });
      return Object.fromEntries(groups);
    });

    // Список препаратов для каждой аудитории
    const audienceMedications = computed(() => {
      const medsByAudience = {};
      products.value.forEach(product => {
        let audiences = Array.isArray(product.ufCrm26_1753110034) ? 
          product.ufCrm26_1753110034 : 
          [product.ufCrm26_1753110034];
        audiences.forEach(audienceId => {
          const audienceTitle = audienceTitles.value.get(String(audienceId)) || String(audienceId);
          if (!medsByAudience[audienceTitle]) {
            medsByAudience[audienceTitle] = [];
          }
          if (!medsByAudience[audienceTitle].some(med => med.id === product.id)) {
            medsByAudience[audienceTitle].push({ id: product.id, title: product.title });
          }
        });
      });
      return medsByAudience;
    });

    // Список оборудования для каждой аудитории
    const audienceEquipment = computed(() => {
      const equipByAudience = {};
      equipment.value.forEach(equip => {
        let audiences = Array.isArray(equip.ufCrm62_1753257668) ? 
          equip.ufCrm62_1753257668 : 
          [equip.ufCrm62_1753257668];
        audiences.forEach(audienceId => {
          const audienceTitle = audienceTitles.value.get(String(audienceId)) || String(audienceId);
          if (!equipByAudience[audienceTitle]) {
            equipByAudience[audienceTitle] = [];
          }
          if (!equipByAudience[audienceTitle].some(e => e.id === equip.id)) {
            equipByAudience[audienceTitle].push({ id: equip.id, title: equip.title });
          }
        });
      });
      return equipByAudience;
    });

    // Контакты без целевой аудитории
    const contactsWithoutAudience = computed(() => {
      return contacts.value.filter(contact => !contact.UF_CRM_1753109871 || contact.UF_CRM_1753109871.length === 0);
    });

    onMounted(() => {
      fetchData();
    });

    return {
      loading,
      error,
      contacts,
      domain,
      groupedByAudience,
      contactsWithoutAudience,
      audienceTitles,
      cityTitles,
      audienceMedications,
      audienceEquipment,
      deleteCard,
    };
  }
}
</script>
<style>
#app {
    font-size: .8rem;
    padding-bottom: .2rem
}

.text-h6 {
    font-weight: 700;
    margin-bottom: 4px
}

a {
    color: #000
}

.name {
    font-size: .7em!important;
    font-weight: 400
}

.v-list-item-subtitle {
    overflow: visible;
    display: block;
    font-size: 1em!important
}

.medication {
    white-space: nowrap
}

span {
    font-size: .9em
}

.v-list {
    padding: 0
}

.v-list-item--density-default:not(.v-list-item--nav).v-list-item--one-line {
    padding-inline:.5rem}

.v-list .v-list-item--density-default:not(.v-list-item--nav).v-list-item--one-line {
    padding-inline:0;padding: 0 .5rem
}

.v-card-title {
    padding: .5rem;
    padding-bottom: 0;
    color: gray;
    font-size: 1.35em;
    font-weight: 600
}

.loading {
    width: 100vw;
    height: 100vh;
    position: fixed;
    top: 0;
    left: 0;
    background-color: #fff;
    z-index: 1000;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 2rem;
    flex-direction: column;
    gap: 2rem
}

.item {
    margin: .4rem;
    color: 000000
}

.v-list-item-subtitle span {
    font-size: 1em
}

.delete{
  transform: scale(0.5);
  position: relative;
  left: -0.5rem;
}


</style>