<template>
  <v-row>
    <v-col>
      <v-card elevation="7">
        <v-card class="pa-2" color="#424242" dark>Plantillas de Campañas</v-card>
        <div class="pa-4">
          <v-select v-model="item" :items="items" label="Plantillas Aprobadas" outlined
            @change="selectTemplate"></v-select>

          <template v-if="template">
            <v-divider class="mb-4"></v-divider>
            <v-alert v-if="template.status !== 'APPROVED'" type="error">Plantilla no aprobada y no se puede utilizar para
              enviar mensajes.</v-alert>

            <v-container>
              <v-tabs v-model="tab" background-color="teal darken-1" dark>
                <v-tab @click="updateRecipients">Recipiente de Contactos</v-tab>
                <v-tab @click="updateRecipients">Tabla de Contactos</v-tab>

                <v-tabs-items v-model="tab">
                  <!-- Pestaña 1: Recipiente de Contactos -->
                  <v-tab-item>
                    <v-card>
                      <div class="my-5">
                        <h5 class="text-h5">Recipiente de contactos</h5>
                        <v-textarea v-model="recipients" outlined hint="Introduzca un destinatario por línea."
                          persistent-hint @input="limitAndBreakLines">
                        </v-textarea>
                      </div>
                    </v-card>
                  </v-tab-item>

                  <!-- Pestaña 2: Tabla de Contactos -->
                  <v-tab-item>
                    <v-card>
                      <div class="my-5">
                        <h5 class="text-h5">Tabla de Contactos</h5>
                        <v-text-field v-model="filter" label="Filtrar por nombre" outlined></v-text-field>
                        <div class="table-container">
                          <table class="styled-table">
                            <thead>
                              <tr>
                                <th>#</th>
                                <th>Nombre</th>
                                <th>Celular</th>
                                <th>Seleccionar</th>
                              </tr>
                            </thead>
                            <tbody>
                              <tr v-for="(user, index) in paginatedUsers" :key="index">
                                <td>{{ (currentPage - 1) * pageSize + index + 1 }}</td>
                                <td>{{ user.name }}</td>
                                <td>{{ user.phone }}</td>
                                <td>
                                  <v-checkbox @click="updateRecipients" v-model="user.isSelected" label="Enviar" color="primary"></v-checkbox>
                                </td>
                              </tr>
                            </tbody>
                          </table>
                        </div>
                        <v-pagination v-model="currentPage" :length="totalPages"></v-pagination>
                      </div>
                    </v-card>
                  </v-tab-item>
                </v-tabs-items>
              </v-tabs>
            </v-container>



            <div v-if="template.header" class="my-5">
              <h5 class="text-h5">Header</h5>
              <p v-if="template.header.format == 'TEXT'">
                {{ template.header.text }}
              </p>
              <v-text-field v-else v-model="header_url" outlined class="mt-2"
                :label="`${template.header.format} URL`"></v-text-field>
            </div>

            <div v-if="template.body" class="my-5">
              <h5 class="text-h5">Body</h5>
              <p class="pre-wrap">{{ template.body }}</p>
              <v-text-field v-for="(placeholder, index) in template.body_placeholders" :key="index"
                v-model="body_placeholders[index]" outlined :label="placeholder.text"></v-text-field>
            </div>

            <div v-if="template.footer" class="my-5">
              <h5 class="text-h5">Footer</h5>
              <p class="pre-wrap">{{ template.footer }}</p>
            </div>

            <div v-if="template.buttons" class="my-3">
              <h5 class="text-h5">Buttons</h5>
              <ul>
                <li v-for="(button, index) in template.buttons" :key="index">
                  {{ button.text }} <em>({{ button.type }})</em>
                </li>
              </ul>
            </div>

            <div class="mt-8">
              <v-btn block large color="teal darken-1" dark :loading="sending" @click="send">Enviar</v-btn>
            </div>
          </template>
        </div>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
export default {
  data: () => ({
    item: '',
    tab: null,
    template: null,
    templates: [],
    items: [],
    recipients: [],
    body_placeholders: [],
    header_url: null,
    sending: false,
    filter: '', // Variable para almacenar el filtro por nombre
    pageSize: 10,
    currentPage: 1,
    users: [],
  }),
  watch: {
    recipients(newValue) {
      this.concatenate591(newValue);
    }
  },
  computed: {
    filteredUsers() {
      // Filtrar usuarios basándose en el nombre
      return this.users.filter(user => user.name.toLowerCase().includes(this.filter.toLowerCase()));
    },

    totalPages() {
      return Math.ceil(this.filteredUsers.length / this.pageSize);
    },
    paginatedUsers() {
      const startIndex = (this.currentPage - 1) * this.pageSize;
      const endIndex = startIndex + this.pageSize;
      return this.filteredUsers.slice(startIndex, endIndex);
    },
  },
  created() {
    this.loadTemplates();
    this.fetchUsers();  // Llamar al método para obtener los usuarios al cargar el componente
  },
  methods: {
    concatenate591(value) {
      let recipientsArray = value.split('\n');

      recipientsArray = recipientsArray.map(recipient => {
        if (!recipient.includes("591")) {
          return "591" + recipient;
        }
        return recipient;
      });

      this.recipients = recipientsArray.join('\n');
    },
    limitAndBreakLines() {
      let recipientsArray = this.recipients.split('\n');

      recipientsArray = recipientsArray.map(recipient => {
        // Limitar a 11 caracteres y agregar salto de línea si es necesario
        const chunks = recipient.match(/.{1,11}/g);
        return chunks ? chunks.join('\n') : ''; // Agregar un salto de línea si se superan los 8 caracteres
      });

      this.recipients = recipientsArray.join('\n');
    },
    loadTemplates() {
      this.$axios.get('/message-templates').then(({ data }) => {
        this.items = data.data.map((template, index) => {
          this.templates.push(template)
          return {
            text: `${template.name} (${template.language})`,
            value: index,
          }
        })
      })
    },
    selectTemplate() {
      this.template = this.templates[this.item]
      this.body_placeholders = []
      this.header_url = null
      this.formatTemplate()
    },
    send() {
      this.sending = true
      const payload = {
        recipients: this.recipients,
        header_url: this.header_url,
        header_type: this.template.header?.format || null,
        body_placeholders: this.body_placeholders,
        template_name: this.template.name,
        template_language: this.template.language,
      }
      this.$axios
        .post('/send-message-templates', payload)
        .then(({ data }) => {
          alert('Message(s) successfully sent!')
        })
        .catch((err) => {
          alert(err)
        })
        .finally(() => {
          this.sending = false
        })
    },
    formatTemplate() {
      this.template.header = null
      this.template.body = null
      this.template.footer = null
      this.template.buttons = null
      this.template.components.forEach((element) => {
        if (element.type === 'HEADER') this.template.header = element
        else if (element.type === 'BODY') {
          this.template.body = element.text
          this.template.body_placeholders = this.findPlaceholders(element.text)
        } else if (element.type === 'FOOTER')
          this.template.footer = element.text
        else if (element.type === 'BUTTONS')
          this.template.buttons = element.buttons
      })
    },
    findPlaceholders(text) {
      const regexp = /{{(.*?)}}/g
      const exec = text.matchAll(regexp)
      const matches = []

      for (const match of exec) {
        matches.push({ text: match[0], value: '' })
      }
      return matches
    },
    fetchUsers() {
      // Llamar a la API para obtener los datos de los usuarios
      // Asegúrate de ajustar la ruta según tu configuración
      this.$axios.get('/get-users')
        .then(response => {
          this.users = response.data.data;
        })
        .catch(error => {
          console.error('Error al obtener los usuarios:', error);
        });
    },
    updateRecipients() {
      if (this.tab === 1) {
        // Actualizar recipients solo si estamos en la pestaña de Recipientes
        this.recipients = this.getSelectedRecipients();
      }
    },

    getSelectedRecipients() {
      const selectedUsers = this.filteredUsers.filter(user => user.isSelected);
      const selectedNumbers = selectedUsers.map(user => user.phone);
      return selectedNumbers.join('\n');
    },
  },
}
</script>

<style>
.table-container {
  overflow-x: auto;
}

.styled-table {
  width: 100%;
  border-collapse: collapse;
  margin: 25px 0;
  font-size: 18px;
  text-align: left;
}

.styled-table th,
.styled-table td {
  padding: 12px 15px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

.styled-table th {
  background-color: #f8f8f8;
  font-size: 16px;
  font-weight: bold;
  color: #333;
}

.styled-table tbody tr:hover {
  background-color: #f5f5f5;
}
</style>