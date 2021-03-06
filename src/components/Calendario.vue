<template>
  <v-row class="fill-height">
    <v-col>
      <v-sheet height="64">
        <v-toolbar
          flat
        >
          <v-btn color="primary" dark class="mr-4" v-on:click="dialog = true">
            CRIAR EVENTO
          </v-btn>
          <v-btn
            outlined
            class="mr-4"
            color="grey darken-2"
            @click="setToday"
          >
            Hoje
          </v-btn>
          <v-btn
            fab
            text
            small
            color="grey darken-2"
            @click="prev"
          >
            <v-icon small>
              mdi-chevron-left
            </v-icon>
          </v-btn>
          <v-btn
            fab
            text
            small
            color="grey darken-2"
            @click="next"
          >
            <v-icon small>
              mdi-chevron-right
            </v-icon>
          </v-btn>
          <v-toolbar-title v-if="$refs.calendar">
            {{ $refs.calendar.title }}
          </v-toolbar-title>
          <v-spacer></v-spacer>
          <v-menu
            bottom
            right
          >
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                outlined
                color="grey darken-2"
                v-bind="attrs"
                v-on="on"
              >
                <span>{{ typeToLabel[type] }}</span>
                <v-icon right>
                  mdi-menu-down
                </v-icon>
              </v-btn>
            </template>
            <v-list>
              <v-list-item @click="type = 'day'">
                <v-list-item-title>Dia</v-list-item-title>
              </v-list-item>
              <v-list-item @click="type = 'week'">
                <v-list-item-title>Semana</v-list-item-title>
              </v-list-item>
              <v-list-item @click="type = 'month'">
                <v-list-item-title>Mês</v-list-item-title>
              </v-list-item>
              <v-list-item @click="type = '4day'">
                <v-list-item-title>4 dias</v-list-item-title>
              </v-list-item>
            </v-list>
          </v-menu>
        </v-toolbar>
      </v-sheet>
      <v-sheet height="600">
        <v-calendar
          ref="calendar"
          v-model="focus"
          color="primary"
          :events="events"
          :event-color="getEventColor"
          :type="type"
          @click:event="showEvent"
          @click:more="viewDay"
          @click:date="viewDay"
          @change="updateRange"
          :weekday="[0, 1, 2, 3, 4, 5, 6]"
          locale="pt-br"
          :short-weekdays="true"
        ></v-calendar>
        <v-dialog v-model="dialog" max-width="700">
            <v-card>
                <v-conteiner>
                    <v-form v-on:submit.prevent="adicionarEvento">
                        <v-text-field 
                           type="text" 
                           label="Nome" 
                           v-model="name"
                        >
                        </v-text-field>
                        <v-text-field 
                           type="text" 
                           label="Descrição" 
                           v-model="details"
                        >
                        </v-text-field>
                        <v-text-field 
                           type="date" 
                           label="Início do evento" 
                           v-model="start"
                        >
                        </v-text-field>
                        <v-text-field 
                           type="date" 
                           label="Fim do evento" 
                           v-model="end"
                        >
                        </v-text-field>
                        <v-text-field 
                           type="color" 
                           label="Cor do evento" 
                           v-model="color"
                        >
                        </v-text-field>
                        <v-btn 
                            type="submit" 
                            color="primary" 
                            class="mr-4" 
                            v-on:click.stop="dialog = false"
                        >
                            Definir
                        </v-btn>
                    </v-form>
                </v-conteiner>
            </v-card>
        </v-dialog>
        <v-menu
          v-model="selectedOpen"
          :close-on-content-click="false"
          :activator="selectedElement"
          offset-x
        >
          <v-card
            color="grey lighten-4"
            min-width="350px"
            flat
          >
            <v-toolbar
              :color="selectedEvent.color"
              dark
            >
              <v-btn icon v-on:click="deletarEvento(selectedEvent)">
                <v-icon>mdi-delete</v-icon>
              </v-btn>
              <v-toolbar-title v-html="selectedEvent.name"></v-toolbar-title>
              <v-spacer></v-spacer>
            </v-toolbar>
            <v-card-text>
              <v-form v-if="currentlyEditing !== selectedEvent.id">
                <strong>Nome:</strong> {{selectedEvent.name}}
                <br>
                <strong>Descrição:</strong> {{selectedEvent.details}}
                <br>
                <strong>Início:</strong> {{selectedEvent.start}}
                <br>
                <strong>Fim:</strong> {{selectedEvent.end}}
              </v-form>
              <v-form v-else>
                <v-text-field 
                  type="text" 
                  v-model="selectedEvent.name"
                  label="Editar nome"
                >
                </v-text-field>
                <textarea-autosize
                  type="text"
                  v-model="selectedEvent.details"
                  style="width: 100%"
                  :min-height="100"
                ></textarea-autosize>
                <v-text-field
                  type="date"
                  v-model="selectedEvent.start"
                  label="Editar data de início"
                ></v-text-field>
                <v-text-field
                  type="date"
                  v-model="selectedEvent.end"
                  label="Editar data de fim"
                ></v-text-field>
              </v-form>
            </v-card-text>
            <v-card-actions>
              <v-btn 
                text 
                v-on:click.prevent="editarEvento(selectedEvent.id)"
                v-if="currentlyEditing !== selectedEvent.id"
              > 
                Editar
              </v-btn>
              <v-btn 
                text 
                v-else 
                v-on:click.prevent="atualizarEvento(selectedEvent)"
              >
                Salvar alterações
              </v-btn>
              <v-btn
                text
                color="secondary"
                @click="selectedOpen = false"
              >
                Cancelar
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-menu>
      </v-sheet>
    </v-col>
  </v-row>
</template>

<script>
  import Eventos from '../services/eventos'

  export default {
    name: 'Calendario',
    data: () => ({
      focus: new Date().toISOString().substr(0, 10),
      type: 'month',
      typeToLabel: {
        month: 'Mês',
        week: 'Semana',
        day: 'Dia',
        '4day': '4 dias',
      },
      selectedEvent: {},
      selectedElement: null,
      selectedOpen: false,
      events: [],
      id: null,
      name: null,
      details: null,
      start: null,
      end: null,
      color: '#1976D2',
      dialog: false,
      currentlyEditing: null
    }),
    mounted () {
      this.$refs.calendar.checkChange()
    },
    created() {
        this.getEvento();
    },
    methods: {
      async getEvento() {
          try {
            await Eventos.listar().then(resposta => {
              this.events = [];
              const snapshot = resposta.data;
              snapshot.forEach(e => {
                this.events.push({
                  id: e.id,
                  name: e.name,
                  details: e.details,
                  start: e.start.substr(0, 10),
                  end: e.end.substr(0, 10),
                  color: e.color
                });
              });
            });
          } catch (error) {
              console.log(error);
          }
      },
      async adicionarEvento() {
          try {
              if (this.name && this.details && this.start && this.end && this.color) {
                await Eventos.registrar({
                  name: this.name,
                  details: this.details,
                  start: this.start,
                  end: this.end,
                  color: this.color
                }).then(resposta => {
                  console.log(resposta.data);
                });
                this.getEvento();
                this.name = null;
                this.details = null;
                this.start = null;
                this.end = null;
                this.color = '#1976D2';
              } else {
                console.log('Campo obligatório');
              }
          } catch (error) {
              console.log(error);
          }
      },
      async deletarEvento(e) {
        try {
          await Eventos.apagar(e).then(resposta => {
            console.log(resposta.data);
            this.selectedOpen = false;
            this.getEvento();
          });
        } catch (error) {
          console.log(error);
        }
      },
      async editarEvento(idEvento) {
        this.currentlyEditing = idEvento;
      },
      async atualizarEvento(e) {
        try {
          await Eventos.atualizar(e).then(resposta => {
            console.log(resposta.data);
          });
          this.selectedOpen = false;
          this.currentlyEditing = null;
        } catch (error) {
          console.log(error);
        }
      },
      viewDay ({ date }) {
        this.focus = date
        this.type = 'day'
      },
      getEventColor (event) {
        return event.color
      },
      setToday () {
        this.focus = new Date().toISOString().substr(0, 10);
      },
      prev () {
        this.$refs.calendar.prev()
      },
      next () {
        this.$refs.calendar.next()
      },
      showEvent ({ nativeEvent, event }) {
        const open = () => {
          this.selectedEvent = event
          this.selectedElement = nativeEvent.target
          setTimeout(() => {
            this.selectedOpen = true
          }, 10)
        }

        if (this.selectedOpen) {
          this.selectedOpen = false
          setTimeout(open, 10)
        } else {
          open()
        }

        nativeEvent.stopPropagation()
      },
      updateRange ({ start, end }) {
        const events = []

        const min = new Date(`${start.date}T00:00:00`)
        const max = new Date(`${end.date}T23:59:59`)
        const days = (max.getTime() - min.getTime()) / 86400000
        const eventCount = this.rnd(days, days + 20)

        for (let i = 0; i < eventCount; i++) {
          const allDay = this.rnd(0, 3) === 0
          const firstTimestamp = this.rnd(min.getTime(), max.getTime())
          const first = new Date(firstTimestamp - (firstTimestamp % 900000))
          const secondTimestamp = this.rnd(2, allDay ? 288 : 8) * 900000
          const second = new Date(first.getTime() + secondTimestamp)

          events.push({
            name: this.names[this.rnd(0, this.names.length - 1)],
            start: first,
            end: second,
            color: this.colors[this.rnd(0, this.colors.length - 1)],
            timed: !allDay,
          })
        }

        this.events = events
      },
      rnd (a, b) {
        return Math.floor((b - a + 1) * Math.random()) + a
      },
    },
  }
</script>