<template>
<div class="row">

  <div class="col-md-12">
    <div class="card card-plain">
      <paper-table @view="handleView" @create="handleCreate" @delete="handleDelete" @update="handleUpdate" :subData="clientes" type="hover" :title="table.title" :sub-title="table.subTitle" :sort="sort" :data="reservas" :columns="table.columns">

      </paper-table>
      <div v-if="reserva_isCreate" class="reserva" :class="reserva_class">
        <div class="reserva__header">
          <div class="reserva--column">
            <span class="reserva__title">Criação de Reserva</span>
          </div>
          <span class="ti-close reserva__close" @click.stop="resetFields($event)"></span>
        </div>
        <div class="reserva__body">
          <div class="reserva__input-group">
            <label for="">NOME</label>
            <v-select v-model="reserva.cliente.nome" :options="nomeClientes"></v-select>
          </div>
          <div class="reserva__input-group">
            <label for="">NUMERO DA QUADRA</label>
            <input type="number" max="10" min="0" v-model="reserva.quadra">
          </div>
          <div class="reserva__input-group">
            <label for="">NUMERO DE RESERVAS</label>
            <input type="number" max="10" min="0" v-model="reserva.reservas">
          </div>
          <div class="reserva__input-group">
            <label for="">RESERVADO PARA</label>
            <flat-pickr :config="config" v-model="reserva.dataReservada"></flat-pickr>
          </div>
                  <div class="reserva__input-group">
              <label for="">QUANTIDADE</label>
              <input type="number" v-model="reserva.quantidade">
            </div>
          <div class="reserva__input-group">
            <label>PAGO?
            <input type="checkbox" v-model="reserva.pagamento" >
            </label>
          </div>
          <div class="reserva__button-group">
            <button @click.prevent="handleCreate($event)">Enviar</button>
            <button @click.prevent="resetFields($event)">Limpar</button>
          </div>
        </div>
      </div>
      <div v-else class="reserva" :class="reserva_class">
        <div class="reserva__header">
          <div class="reserva--column">
            <span class="reserva__title">Edição de Reserva</span>
          </div>
          <span class="ti-close reserva__close" @click.stop="resetFields($event)"></span>
        </div>
        <div class="reserva__body">
          <div class="flex flex-row">
            <div class="reserva__input-group">
              <label for="">NOME</label>
              <input type="text" v-model="reserva.cliente.nome">
            </div>
            <div class="reserva__input-group">
              <label for="">E-MAIL</label>
              <input type="text" v-model="reserva.cliente.email">
            </div>
          </div>
          <div class="reserva__input-group">
            <label for="">CPF</label>
            <input type="text" v-mask="['###.###.###-##']" v-model="reserva.cliente.cpf">
          </div>
          <div class="flex flex-row">

            <div class="reserva__input-group">
              <label for="">SALDO ATUAL</label>
              <input type="text" v-mask="['R$ ###,00']" v-model="reserva.cliente.saldo">
            </div>
            <div class="reserva__input-group">
              <label for="">DATA DO PAGAMENTO</label>
              <input type="text" v-model="reserva.dataPagamento">
            </div>
          </div>
          <div class="flex flex-row">
            <div class="reserva__input-group">
              <label for="">NUMERO DA QUADRA</label>
              <input type="text" v-model="reserva.quadra">
            </div>
            <div class="reserva__input-group">
              <label for="">RESERVADO PARA</label>
              <input type="text" v-model="reserva.dataReservada">
            </div>
            <div class="reserva__input-group">
              <label for="">QUANTIDADE</label>
              <input type="number" v-model="reserva.quantidade">
            </div>
          </div>
          <div class="reserva__input-group">
            <label for="">CRIADO EM</label>
            <input type="text" v-model="reserva.created_at">
          </div>
          <div class="reserva__button-group">
            <button>Enviar</button>
            <button @click.prevent="resetFields($event)">Limpar</button>
          </div>
        </div>

      </div>
    </div>
  </div>

</div>
</template>
<script>
import PaperTable from "components/UIComponents/PaperTable.vue";
import vSelect from 'vue-select'
import flatPickr from 'vue-flatpickr-component';
import 'flatpickr/dist/flatpickr.css';
const tableColumns = [{
    name: "Nome do Cliente",
    type: "select"
  },
  {
    name: "Dia",
    type: "datetime",
    mask: "##/##/#### ##:##"
  },
  {
    name: "Reservas",
    type: "number"
  },
  {
    name: "Pago",
    type: "text"
  },
  {
    name: "Quadra",
    type: "number"
  }
];

export default {
  created() {
    // this.$Progress.start()
    this.$store.dispatch("load-reservas");
    this.$store.dispatch("load-clientes");
  },
  components: {
    PaperTable,
    vSelect,
    flatPickr
  },
  computed: {
    reservas() {
      return this.$store.state.reservas;
    },
    clientes() {
      return this.$store.state.clientes
    },
    nomeClientes() {
      return this.$store.state.clientes.map((value, index) => value['nome do cliente'])
    }
  },
  methods: {
    async handleUpdate(payload) {
      this.$Progress.start();
      payload.data.column = payload.data.column.replace(
        "Nome do Cliente",
        "nome"
      );
      payload.data.column = payload.data.column.replace("Dia", "dataReservada");
      payload.data.column = payload.data.column.replace(
        "Reservas",
        "numReservas"
      );
      let data = {};
      if (payload.data.column == "nome") {
        data = {
          cliente: {
            nome: payload.data.value
          }
        };
      } else {
        data[payload.data.column] = payload.data.value;
      }
      await this.$http
        .put(`//websports.herokuapp.com/api/reservas/${payload.data.id}`, data)
        .then(response => {
          this.$Progress.finish();
        })
        .catch(err => {
          this.$Progress.fail();
        });
      payload.e.target.setAttribute("readonly", "true");
      this.$store.dispatch("load-reservas");
    },
    async handleCreate(payload = '') {
      if (payload.target.localName == "button") {
        this.$Progress.start();
        console.log(this.reserva)
        await this.$http
          .post(`//websports.herokuapp.com/api/reservas`, this.reserva)
          .then(response => {
            this.$Progress.finish();
          })
          .catch(err => {
            console.error(err)
            this.$Progress.fail();
          });
        return
      }

      this.reserva_isCreate = true;
      this.reserva_class = "reserva--active"

    },
    async handleView(payload) {
      this.reserva_isCreate = false;
      this.$Progress.start();
      await this.$http
        .get(`//websports.herokuapp.com/api/reservas/${payload}`)
        .then(res => {
          this.reserva = {
            cliente: {
              cpf: res.data.cliente.cpf,
              email: res.data.cliente.email,
              nome: res.data.cliente.nome,
              saldo: res.data.cliente.saldo
            },
            dataReservada: res.data.dataReservada,
            dataPagamento: res.data.pagamento.dataPagamento,
            quadra: res.data.quadra.id,
            created_at: res.data.created_at
          };
          this.$Progress.finish();
        })
        .catch(err => {
          this.$Progress.fail();
          console.error(err);
        });
      this.reserva_class = "reserva--active";
      this.$store.dispatch("load-reservas");
    },
    async handleDelete(payload) {
      this.$Progress.start();
      await this.$http
        .delete(`//websports.herokuapp.com/api/reservas/${payload}`)
        .then(res => {
          this.$Progress.finish();
        })
        .catch(err => {
          this.$Progress.fail();
        });
      this.$store.dispatch("load-reservas");
    },
    resetFields(e) {
      if (e.target.localName != "button") {
        this.reserva_class = "reserva--closed";
      }
      this.reserva = {
        cliente: {
          cpf: "",
          email: "",
          nome: "", 
          saldo: ""
        },
        quantidade: '',
        dataReservada: "",
        dataPagamento: "",
        created_at: "",
        updated_at: "",
        quadra_id: "",
        pagamento_id: "",
        reservas: ""
      };
    }
  },
  mounted() {},
  data() {
    return {
      config: {
        enableTime: true,
        dateFormat: 'd-m-Y H:i',
        locale: 'pt-BR'
      },
      sort: "dia",
      table: {
        title: "Listagem de Reservas",
        subTitle: "Para qualquer alteração, clique duas vezes em cima do registro",
        columns: [tableColumns]
      },
      id: 0,
      reserva_isCreate: true,
      reserva_class: "",
      reserva: {
       cliente: {
          cpf: "",
          email: "",
          nome: "", 
          saldo: ""
        },
        quantidade: '',
        dataReservada: "",
        dataPagamento: "",
        created_at: "",
        updated_at: "",
        quadra_id: "",
        pagamento_id: "",
        reserva: "",
        pagamento: '',
      }
    }
  }
}
</script>
<style lang="scss">
.flex {
  display: flex;
}

.dropdown.v-select {
  width: 100%;
}

.dropdown-toggle {
  border: none !important;
}

.reserva {
  position: fixed;
  height: 100vh;
  right: 0;
  top: 0;
  transform: translateX(35vw);
  display: flex;
  flex-direction: column;
  width: 35vw;
  z-index: 9000;
  background: rgba(255, 255, 255, 0.97);
  padding: 30px;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23);
  &--closed {
    animation: translateX-reverse 0.4s forwards;
  }
  &--active {
    animation: translateX 0.4s forwards;
  }
  &--column {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100%;
  }
  &__header {
    width: 100%;
    display: flex;
    margin-bottom: 40px;
    justify-content: space-between;
  }
  &__title {
    margin: 0 auto;
    font-size: 1.8em;
  }
  &__close {
    cursor: pointer;
    padding: 4px;
    font-size: 1.8em;
  }
  &__input-group {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: baseline;
    border-bottom: 1px solid rgba(50, 50, 50, 0.1);
    margin-bottom: 10px;
    width: 100%;
    label {
      text-align: right;
      margin-right: 20px;
    }
    input {
      text-align: left;
      padding-left: 0;
      color: #444;
      font-weight: 400;
      background: transparent;
      border: none;
    }
    input[type=checkbox] {
      width: auto;
    }
  }
  &__button-group {
    flex: 1;
    display: flex;
    margin-top: 20px;
    button {
      height: 40px;
      width: 100%;
      border: none;
      text-transform: uppercase;
      font-weight: bold;
      color: #666;
      transition: 0.3s;
      &:first-child {
        &:hover {
          background: #ccc;
        }
      }
      &:last-child {
        background: transparent;
        border: 3px solid #ddd;
        &:hover {
          background: #ccc;
          border-color: #ccc;
        }
      }
    }
  }
}

@keyframes translateX {
  0% {
    transform: translateX(35vw);
  }
  100% {
    transform: translateX(0);
  }
}

@keyframes translateX-reverse {
  0% {
    transform: translateX(0);
  }
  100% {
    transform: translateX(35vw);
  }
}
</style>
