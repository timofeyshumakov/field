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
                <v-btn 
                  v-if="enityId === 'CRM_DEAL'" 
                  class="key-person-btn mr-2" 
                  :color="isKeyPerson(contact.ID) ? 'success' : 'grey-lighten-2'" 
                  icon 
                  @click="toggleKeyPerson(contact.ID)"
                  aria-label="Ключевое лицо" 
                  title="Ключевое лицо"
                  size="x-small"
                >
                  <v-icon v-if="isKeyPerson(contact.ID)" size="small">mdi-check</v-icon>
                  <v-icon v-else size="small" style="opacity: 0.5">mdi-account</v-icon>
                </v-btn>
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
                <v-btn 
                  v-if="enityId === 'CRM_DEAL'" 
                  class="key-person-btn mr-2" 
                  :color="isKeyPerson(contact.ID) ? 'success' : 'grey-lighten-2'" 
                  icon 
                  @click="toggleKeyPerson(contact.ID)"
                  aria-label="Ключевое лицо" 
                  title="Ключевое лицо"
                  size="x-small"
                >
                  <v-icon v-if="isKeyPerson(contact.ID)" size="small">mdi-check</v-icon>
                  <v-icon v-else size="small" style="opacity: 0.5">mdi-account</v-icon>
                </v-btn>
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

    <!-- Блок аудиторий без контактов, но с препаратами или оборудованием -->
    <v-card 
      class="mb-3 elevation-1"
      v-for="(audience, audienceName) in audiencesWithoutContacts"
      :key="`no-contacts-${audienceName}`"
    >
      <v-card-title class="secondary white--text">
        {{ audienceTitles.get(audienceName) || audienceName }}
      </v-card-title>
      
      <!-- Список препаратов для данной аудитории -->
      <v-list-item-subtitle class="item" v-if="audience.medications && audience.medications.length > 0">
        <span>Препараты: </span>
        <a
          v-for="(medication, m) in audience.medications" 
          :key="`med-no-contacts-${m}`"
          target="_blank" :href="'https://' + domain + '/page/spravochniki/pgirfw/type/189/details/' + medication.id + '/'"
        >{{ audience.medications.length !== m + 1 ? medication.title + ", " : medication.title }}</a>
      </v-list-item-subtitle>
      
      <!-- Список оборудования для данной аудитории -->
      <v-list-item-subtitle class="item" v-if="audience.equipment && audience.equipment.length > 0">
        <span>Оборудование: </span>
        <a
          v-for="(equipment, e) in audience.equipment" 
          :key="`equip-no-contacts-${e}`"
          class="pl-0"
          target="_blank" :href="'https://' + domain + '/crm/type/1104/details/' + equipment.id + '/'"
        >{{ audience.equipment.length !== e + 1 ? equipment.title + ", " : equipment.title}}</a>
      </v-list-item-subtitle>
    </v-card>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue';
import { callApi, getListElements } from "../functions/callApi";

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
    const companyTargetAudience = ref(new Map()); // Маппинг компания -> целевые аудитории

    // Функция для парсинга поля компании UF_CRM_1756633452
    const parseCompanyTargetAudience = (fieldValue) => {
      const result = new Map();
      if (!fieldValue) return result;
      
      const lines = fieldValue.split('\n').filter(line => line.trim());
      lines.forEach(line => {
        const match = line.match(/^(\d+)\s*-\s*(.+)$/);
        if (match) {
          const companyId = match[1].trim();
          const audienceIds = match[2].split(',')
            .map(id => id.trim())
            .filter(id => id && id !== '');
          
          result.set(companyId, audienceIds);
        }
      });
      return result;
    };
    const keyPersons = ref([]); // Массив ID ключевых лиц

    // Функция для проверки, является ли контакт ключевым лицом
    const isKeyPerson = (contactId) => {
      return keyPersons.value.includes(contactId);
    };

    // Функция для переключения статуса ключевого лица
    const toggleKeyPerson = async (contactId) => {
      try {
          keyPersons.value = contactId;
BX24.callMethod("crm.deal.update", {id: dealId.value, fields: { UF_CRM_1756807710: contactId}},
      () => {});
      } catch (error) {
        console.error('Ошибка при обновлении ключевых лиц:', error);
      }
    };

    // Загрузка ключевых лиц при инициализации
    const fetchKeyPersons = async () => {
      try {
        if (enityId.value === 'CRM_DEAL') {
          const deal = await callApi("crm.deal.list", { 
            ID: dealId.value 
          }, ["UF_CRM_1756807710"]);
          
          if (deal[0] && deal[0].UF_CRM_1756807710) {
            keyPersons.value = deal[0].UF_CRM_1756807710
          }
        }
      } catch (error) {
        console.error('Ошибка при загрузке ключевых лиц:', error);
      }
    };

    const fetchCompanyData = async () => {
      domain.value = BX24.getDomain();
      enityId.value = BX24.placement.info().options.ENTITY_ID;
      let contacts = "";

      try {
        if (enityId.value === "CRM_DEAL") {
          //BX24.placement.info().options.ENTITY_VALUE_ID
          const deal = await callApi("crm.deal.list", { ID: BX24.placement.info().options.ENTITY_VALUE_ID}, ["COMPANY_ID", "UF_CRM_1754290331", "UF_CRM_1753365812", "UF_CRM_1758377551"]);
          dealId.value = deal[0].ID;
          companyId.value = deal[0].COMPANY_ID;
          contacts = deal[0].UF_CRM_1754290331;

          if (deal[0] && deal[0].UF_CRM_1758377551) {
            // Парсим строку вида "16316,16412,19874,17536"
            const hiddenContactIds = deal[0].UF_CRM_1758377551
              .split(',')
              .map(id => parseInt(id.trim()))
              .filter(id => !isNaN(id));
            const hiddenContacts = new Set(hiddenContactIds);
            contacts = contacts.filter(item => !hiddenContacts.has(+item));
            console.log(contacts);
            console.log(hiddenContacts);
          }

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

        // Парсим поле UF_CRM_1756633452 компании для получения целевых аудиторий
        companyTargetAudience.value = parseCompanyTargetAudience(company.UF_CRM_1756633452);

        return [contacts, company.UF_CRM_1745407296, company.UF_CRM_1753257731];
      } catch (err) {
        console.error('Ошибка:', err);
        error.value = err.message;
        throw err;
      }
    };

    const fetchData = async () => {
      try {
        loading.value = true;
        const [users, productsToFetch, equipmentToFetch] = await fetchCompanyData();
        await fetchKeyPersons();
        const userData = await callApi("crm.contact.list", { ID: users }, ["UF_CRM_1750766630", "NAME", "LAST_NAME", "SECOND_NAME", "POST", "TYPE_ID", "EMAIL", "UF_CRM_1753364801", "UF_CRM_1753083765", "UF_CRM_1756633452"]);
        
        products.value = await callApi("crm.item.list", {id: productsToFetch}, ["id", "title", "ufCrm26_1753365041"], 189);
        equipment.value = await callApi("crm.item.list", {id: equipmentToFetch}, ["id", "title", "ufCrm62_1753365319"], 1104);
        
        const productsFields = await getListElements(216, {}, ["ID", "NAME"]);
        
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

    // Функция для проверки, принадлежит ли контакт к целевой аудитории
    const isContactInTargetAudience = (contact) => {
      // Если у контакта нет целевых аудиторий, он не принадлежит
      if (!contact.UF_CRM_1753364801 || contact.UF_CRM_1753364801.length === 0) {
        return false;
      }

      const targetAudiences = enityId.value === 'CRM_DEAL' ? 
        dealAudience.value.map(id => String(id)) : 
        (companyTargetAudience.value.get(String(companyId.value)) || []);
      
      // Если у сделки/компании нет целевых аудиторий, показываем все контакты
      if (targetAudiences.length === 0) {
        return true;
      }
      
      // Проверяем, есть ли у контакта хотя бы одна целевая аудитория, которая есть у сделки/компании
      return contact.UF_CRM_1753364801.some(audienceId => 
        targetAudiences.includes(String(audienceId))
      );
    };

const contactsWithoutAudience = computed(() => {
  // Для компаний показываем контакты без аудитории
  if (enityId.value === 'CRM_COMPANY') {
    return contacts.value.filter(contact => 
      !contact.UF_CRM_1753364801 || 
      contact.UF_CRM_1753364801.length === 0
    );
  }
  
  // Для сделок используем старую логику
  const targetAudiences = dealAudience.value.map(id => String(id));
  
  if (targetAudiences.length === 0) {
    return [];
  }
  
  return contacts.value.filter(contact => 
    !contact.UF_CRM_1753364801 || 
    contact.UF_CRM_1753364801.length === 0
  );
});

const audiencesWithoutContacts = computed(() => {
  const result = {};
  
  // Для компаний получаем все целевые аудитории из препаратов и оборудования
  if (enityId.value === 'CRM_COMPANY') {
    // Собираем все уникальные аудитории из препаратов и оборудования
    const allAudiences = new Set();
    
    // Добавляем аудитории из препаратов
    products.value.forEach(product => {
      let audiences = Array.isArray(product.ufCrm26_1753365041) ? 
        product.ufCrm26_1753365041 : 
        [product.ufCrm26_1753365041];
      audiences.forEach(audienceId => {
        const audienceTitle = audienceTitles.value.get(String(audienceId)) || String(audienceId);
        // Пропускаем аудитории без названия
        if (audienceTitle && audienceTitle.trim() !== '') {
          allAudiences.add(String(audienceId));
        }
      });
    });
    
    // Добавляем аудитории из оборудования
    equipment.value.forEach(equip => {
      let audiences = Array.isArray(equip.ufCrm62_1753365319) ? 
        equip.ufCrm62_1753365319 : 
        [equip.ufCrm62_1753365319];
      audiences.forEach(audienceId => {
        const audienceTitle = audienceTitles.value.get(String(audienceId)) || String(audienceId);
        // Пропускаем аудитории без названия
        if (audienceTitle && audienceTitle.trim() !== '') {
          allAudiences.add(String(audienceId));
        }
      });
    });
    
    // Для каждой аудитории проверяем, есть ли контакты
    allAudiences.forEach(audienceId => {
      const audienceTitle = audienceTitles.value.get(String(audienceId)) || String(audienceId);
      
      // Пропускаем аудитории без названия
      if (!audienceTitle || audienceTitle.trim() === '') {
        return;
      }
      
      // Если в этой аудитории нет контактов
      if (!filteredGroupedByAudience.value[audienceTitle] || 
          filteredGroupedByAudience.value[audienceTitle].length === 0) {
        
        // Проверяем, есть ли препараты или оборудование для этой аудитории
        const hasMedications = filteredAudienceMedications.value[audienceTitle] && 
                              filteredAudienceMedications.value[audienceTitle].length > 0;
        const hasEquipment = filteredAudienceEquipment.value[audienceTitle] && 
                            filteredAudienceEquipment.value[audienceTitle].length > 0;
        
        // Если есть препараты или оборудование, добавляем в результат
        if (hasMedications || hasEquipment) {
          result[audienceTitle] = {
            medications: filteredAudienceMedications.value[audienceTitle] || [],
            equipment: filteredAudienceEquipment.value[audienceTitle] || []
          };
        }
      }
    });
  } else {
    // Для сделок используем существующую логику
    const targetAudiences = dealAudience.value.map(id => String(id));
    
    if (targetAudiences.length === 0) {
      return result;
    }
    
    targetAudiences.forEach(audienceId => {
      const audienceTitle = audienceTitles.value.get(String(audienceId)) || String(audienceId);
      
      // Пропускаем аудитории без названия
      if (!audienceTitle || audienceTitle.trim() === '') {
        return;
      }
      
      if (!filteredGroupedByAudience.value[audienceTitle] || 
          filteredGroupedByAudience.value[audienceTitle].length === 0) {
        
        const hasMedications = filteredAudienceMedications.value[audienceTitle] && 
                              filteredAudienceMedications.value[audienceTitle].length > 0;
        const hasEquipment = filteredAudienceEquipment.value[audienceTitle] && 
                            filteredAudienceEquipment.value[audienceTitle].length > 0;
        
        if (hasMedications || hasEquipment) {
          result[audienceTitle] = {
            medications: filteredAudienceMedications.value[audienceTitle] || [],
            equipment: filteredAudienceEquipment.value[audienceTitle] || []
          };
        }
      }
    });
  }
  
  return result;
});

const filteredAudienceEquipment = computed(() => {
  const equipByAudience = {};
  const targetAudiences = enityId.value === 'CRM_DEAL' ? 
    dealAudience.value.map(id => String(id)) : 
    (companyTargetAudience.value.get(String(companyId.value)) || []);
  
  // Для компаний показываем все оборудование
  const showAll = enityId.value === 'CRM_COMPANY' || targetAudiences.length === 0;
  
  equipment.value.forEach(equip => {
    let audiences = Array.isArray(equip.ufCrm62_1753365319) ? 
      equip.ufCrm62_1753365319 : 
      [equip.ufCrm62_1753365319];
    audiences.forEach(audienceId => {
      const audienceTitle = audienceTitles.value.get(String(audienceId)) || String(audienceId);
      
      // Пропускаем аудитории без названия
      if (!audienceTitle || audienceTitle.trim() === '') {
        return;
      }
      
      if (showAll || targetAudiences.includes(String(audienceId))) {
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
const filteredGroupedByAudience = computed(() => {
  const groups = new Map();
  
  // Для компаний показываем все целевые аудитории
  const showAllForCompany = enityId.value === 'CRM_COMPANY';
  
  contacts.value
    .filter(contact => showAllForCompany || isContactInTargetAudience(contact))
    .forEach(contact => {
      // Пропускаем контакты без целевой аудитории
      if (!contact.UF_CRM_1753364801 || contact.UF_CRM_1753364801.length === 0) {
        return;
      }
      
      contact.UF_CRM_1753364801.forEach(audienceId => {
        const audienceTitle = audienceTitles.value.get(String(audienceId)) || String(audienceId);
        
        // Пропускаем аудитории без названия
        if (!audienceTitle || audienceTitle.trim() === '') {
          return;
        }
        
        const targetAudiencesForThis = enityId.value === 'CRM_DEAL' ? 
          dealAudience.value.map(id => String(id)) :
          (contact.UF_CRM_1756633452 ? 
            parseCompanyTargetAudience(contact.UF_CRM_1756633452).get(String(companyId.value)) || [] :
            companyTargetAudience.value.get(String(companyId.value)) || []);
        
        if (showAllForCompany || targetAudiencesForThis.length === 0 || targetAudiencesForThis.includes(String(audienceId))) {
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
  const targetAudiences = enityId.value === 'CRM_DEAL' ? 
    dealAudience.value.map(id => String(id)) : 
    (companyTargetAudience.value.get(String(companyId.value)) || []);
  
  // Для компаний показываем все препараты
  const showAll = enityId.value === 'CRM_COMPANY' || targetAudiences.length === 0;
  
  products.value.forEach(product => {
    let audiences = Array.isArray(product.ufCrm26_1753365041) ? 
      product.ufCrm26_1753365041 : 
      [product.ufCrm26_1753365041];
    audiences.forEach(audienceId => {
      const audienceTitle = audienceTitles.value.get(String(audienceId)) || String(audienceId);
      
      // Пропускаем аудитории без названия
      if (audienceTitle > 0) {
        return;
      }
      
      if (showAll || targetAudiences.includes(String(audienceId))) {
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


    onMounted(() => {
      BX24.ready(function () {
          BX24.init(function () {
              fetchData();
          });
      });
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
      enityId,
      audiencesWithoutContacts,
      isKeyPerson,
      toggleKeyPerson,
      keyPersons,
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
.key-person-btn {
  width: 24px !important;
  height: 24px !important;
  border-radius: 100% !important;
}

.key-person-btn .v-icon {
  font-size: 16px !important;
}
</style>