<template>
  <div class="q-pa-xs">
    <q-tree
      ref="tree"
      :nodes="Nodes"
      node-key="label"
      icon="add"
      accordion
    >
      <div
        class="row items-center"
        slot="header-root"
        slot-scope="prop"
      >
        <q-card
          style="width: 290px"
          inline
          color="blue-grey-2"
          text-color="primary"
          @click.native="Eventos(prop.node)"
        >
          <blockquote style="border-radius: 3px 0 0 3px;">
            <span class="q-subheading text-weight-bold text-primary text-no-wrap">{{ prop.node.label }}</span>
          </blockquote>
        </q-card>
      </div>
      <div
        slot="header-body"
        slot-scope="prop"
        class="row items-center"
      >
        <q-card
          class="non-selectable"
          color="blue-grey-2"
          text-color="primary"
        >
          <blockquote
            class="text-no-wrap q-pl-none"
            style="border-radius: 3px 0 0 3px;"
          >
            <q-list
              class="no-padding"
              no-border
              dense
            >
              <q-item class="no-padding q-mb-xs">
                <q-item-side class="flex flex-center">
                  <q-icon
                    :name="JSON.parse(prop.node.icon)[0]"
                    color="primary"
                    size="16px"
                  />
                </q-item-side>
                <q-item-main
                  class="q-caption text-weight-bold text-primary"
                  :label="JSON.parse(prop.node.label)[0]"
                />
              </q-item>
              <q-item class="no-padding">
                <q-item-side class="flex flex-center">
                  <q-icon
                    :name="JSON.parse(prop.node.icon)[1]"
                    color="primary"
                    size="16px"
                  />
                </q-item-side>
                <q-item-main
                  class="q-caption text-weight-bold text-primary"
                  :label="`${JSON.parse(prop.node.label)[1]} - ${JSON.parse(prop.node.label)[2]}`"
                />
              </q-item>
            </q-list>
          </blockquote>
        </q-card>
      </div>
    </q-tree>
  </div>
</template>

<script>
import { date } from 'quasar'
export default {
  name: 'Evento',
  props: ['sala', 'modal', 'novo'],
  filters: {
    FormatarData (data) {
      if (date.isValid(data)) {
        return `${date.formatDate(data, 'DD/MM/YYYY')} - ${date.formatDate(data, 'HH:mm')}h`
      }
    }
  },
  watch: {
    modal () {
      this.$refs.tree.collapseAll()
    }
  },
  methods: {
    AprovarReserva (reservaId) {
      this.$axios.patch(`/reservas/${reservaId}`, {
        dataReserva: Date.now()
      }).then(Res => {
        this.$q.notify({
          type: 'positive',
          timeout: 1000,
          message: 'A reserva foi confirmada com sucesso!'
        })
        this.$emit('reserva')
      })
    },
    RemoverReserva (reservaId) {
      this.$axios.delete(`/reservas/${reservaId}`).then(Res => {
        this.$q.notify({
          type: 'positive',
          timeout: 1000,
          message: 'A reserva foi removida com sucesso!'
        })
        this.$emit('reserva')
      })
    },
    Eventos (evento) {
      this.$emit('modal')
      this.$q.actionSheet({
        title: evento.label,
        grid: true,
        actions: evento.reservada
          ? [
            {
              label: 'Remover',
              color: 'negative',
              icon: 'event_busy',
              handler: () => {
                this.RemoverReserva(evento.children[0].reservaId)
              }
            }
          ]
          : [
            {
              label: 'Aprovar',
              color: 'positive',
              icon: 'event_available',
              handler: () => {
                this.AprovarReserva(evento.children[0].reservaId)
              }
            },
            {
              label: 'Remover',
              color: 'negative',
              icon: 'event_busy',
              handler: () => {
                this.RemoverReserva(evento.children[0].reservaId)
              }
            }
          ]
      })
    }
  },
  computed: {
    Nodes () {
      let nodes = []
      this.sala.eventos.map((evento, index) => {
        if (!evento.reservada && this.novo) {
          nodes[index] = { label: evento.nomeEvento, header: 'root', children: [], reservada: evento.reservada }
          evento.dataEvento.map(reserva => {
            nodes[index].children.push({
              icon: `["far fa-calendar-check", "far fa-clock"]`,
              label: `["${date.formatDate(reserva.dataFinal, 'DD/MM/YYYY')}", "${date.formatDate(reserva.dataInicial, 'HH:mm')}h", "${date.formatDate(reserva.dataFinal, 'HH:mm')}h"]`,
              header: 'body',
              reservaId: evento.reservaId
            })
          })
        } else if (evento.reservada && !this.novo) {
          nodes[index] = { label: evento.nomeEvento, header: 'root', children: [], reservada: evento.reservada }
          evento.dataEvento.map(reserva => {
            nodes[index].children.push({
              icon: `["far fa-calendar-check", "far fa-clock"]`,
              label: `["${date.formatDate(reserva.dataFinal, 'DD/MM/YYYY')}", "${date.formatDate(reserva.dataInicial, 'HH:mm')}h", "${date.formatDate(reserva.dataFinal, 'HH:mm')}h"]`,
              header: 'body',
              reservaId: evento.reservaId
            })
          })
        }
      })
      return nodes
    }
  }
}
</script>
