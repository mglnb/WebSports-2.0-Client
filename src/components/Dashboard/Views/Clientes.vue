<template>
  <div class="row">

    <div class="col-md-12">
      <div class="card card-plain">
        <paper-table @view="handleView" @delete="handleDelete" @create="cliente_class = 'cliente--active'" @update="handleUpdate" type="hover" :title="table.title" :sub-title="table.subTitle" :data="clientes" :columns="table.columns">

        </paper-table>
        <div class="cliente" :class="cliente_class">
          <div class="cliente__header">
            <div class="cliente--column">
              <span class="cliente__title">Criação de cliente</span>
            </div>
            <span class="ti-close cliente__close" @click.stop="cliente_class = 'cliente--closed'"></span>
          </div>
          <div class="cliente__body">
            <div class="cliente__input-group">
              <label for="">NOME</label>
              <input type="text" v-model="cliente.nome">
            </div>
            <div class="flex flex-row">

              <div class="cliente__input-group">
                <label for="">E-MAIL</label>
                <input type="text" v-model="cliente.email">
              </div>
              <div class="cliente__input-group">
                <label for="">CPF</label>
                <input type="text" v-model="cliente.cpf" >
              </div>
            </div>
            <div class="cliente__input-group">
              <label for="">SALDO</label>
              <input type="text" v-model="cliente.saldo" >
            </div>
            <div class="flex flex-row">

            <div class="cliente__input-group">
              <label for="">CEP</label>
              <input type="text" @blur="findCep" v-mask="['#####-###']" v-model="cliente.endereco.cep">
            </div>
              <div class="cliente__input-group">
                <label for="">CIDADE</label>
                <input type="text"  v-model="cliente.endereco.cidade">
              </div>
            </div>
            <div class="cliente__input-group">
              <label for="">Endereço</label>
              <div class="flex flex-row">
                <input type="text" placeholder="Rua" v-model="cliente.endereco.rua">
                <input type="text" placeholder="Numero" v-model="cliente.endereco.numero">
                <input type="text" placeholder="Complemento" v-model="cliente.endereco.complemento">
              </div>
            </div>
            <div class="reserva__button-group">
              <button @click.prevent="handleCreate($event)">Enviar</button>
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
  import axios from 'axios'
  const tableColumns = [{
      name: "Nome do Cliente"
    }, {
      name: 'Email',
      type: 'email'
    }, {
      name: "Endereço"
    },
    {
      name: "CPF",
      mask: '###.###.###-##'
    }, {
      name: "Saldo",
      mask: 'R$ ###,##'
    }
  ];

  export default {
    data() {
      return {
        cliente_class: '',
        table: {
          title: "Listagem de Clientes",
          subTitle: "Para qualquer alteração, clique duas vezes em cima do registro",
          columns: [tableColumns]
        },
        cliente: {
          nome: '',
          email: '',
          endereco: {
            rua: '',
            numero: '',
            complemento: '',
            cidade: '',
            cep: ''
          },
          cpf: '',
          saldo: ''
        }
      };
    },
    created() {
      // this.$Progress.start()
      this.$store.dispatch("load-clientes");
    },
    components: {
      PaperTable
    },

    computed: {
      clientes() {
        return this.$store.state.clientes;
      }
    },
    methods: {
      async handleUpdate(payload) {
        this.$Progress.start()
        payload.data.column = payload.data.column.replace(
          "Nome do Cliente",
          "nome"
        );
        let data;
        if (payload.data.column == "Endereço") {
          payload.data.rua = payload.data.value.split(",")[0];
          payload.data.numero = payload.data.value.split(",")[1].trim();
          payload.data.complemento =
            payload.data.value.split(",")[2].trim() || "";
          data = {
            endereco: {
              rua: payload.data.rua,
              numero: payload.data.numero,
              complemento: payload.data.complemento
            }
          }
        } else {
          data = {
            [payload.data.column.toLowerCase()]: payload.data.value
          }
        }

        await this.$http.put('//websports.herokuapp.com/api/clientes/' + payload.data.id, data)
          .then((response) => {
            this.$Progress.finish()
          })
          .catch((err) => {
            this.$Progress.fail()
          })

        payload.e.target.setAttribute('readonly', 'true')
        this.$store.dispatch("load-clientes");
      },
      async handleCreate(payload) {
        this.$Progress.start()
        await this.$http.post('//websports.herokuapp.com/api/clientes', this.cliente)
          .then(res => {
            this.$Progress.finish()
          })
          .catch(err => {
            this.$Progress.fail()
          })
        this.$store.dispatch("load-clientes");
      },
      async handleDelete(payload) {
        this.$Progress.start()
        await this.$http.delete(`//websports.herokuapp.com/api/clientes/${payload}`)
          .then(res => {
            this.$Progress.finish()
          })
          .catch(err => {
            this.$Progress.fail();
            console.log(err)
          })
        this.$store.dispatch("load-clientes")
      },
      async handleView(payload) {
        this.$Progress.start()

        await this.$http.get(`//websports.herokuapp.com/api/clientes/${payload}`)
          .then(res => {
            this.cliente = {
              nome: res.data.nome || '',
              email: res.data.email || '',
              cep: '',
              endereco: {
                rua: res.data.endereco.rua || '',
                numero: res.data.endereco.numero || '',
                complemento: res.data.endereco.complemento || ''
              },
              cpf: res.data.cpf || '',
              saldo: res.data.saldo || '',
            }
            this.$Progress.finish()
          })
          .catch(err => {
            this.$Progress.fail();
            console.log(err)
          })
        this.cliente_class = "cliente--active"

      },
      findCep() {
        axios.get(`http://viacep.com.br/ws/${this.cliente.endereco.cep}/json/`)
          .then(response => {
            this.cliente.endereco = {
              rua: response.data.logradouro,
              complemento: response.data.complemento,
              numero: '',
              cep: this.cliente.endereco.cep,
              cidade: response.data.localidade
            }
          })
      }
    },
    mounted() {}
  };
</script>

<style lang="scss">
  .cliente {
    position: fixed;
    height: 100vh;
    right: 0;
    top: 0;
    transform: translateX(35vw);
    display: flex;
    flex-direction: column;
    width: 35vw;
    z-index: 9999;
    background: rgba(255, 255, 255, 0.97);
    padding: 30px;
    box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23);
    .flex {
      display: flex;
      &.flex-row {
        flex-direction: row;

      }
    }
    &--closed {
      animation: translateX-reverse .4s forwards;
    }
    &--active {
      animation: translateX .4s forwards;
    }
    &--column {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      width: 100%
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
      display: flex;
      flex-direction: column;
      align-items: baseline;
      border-bottom: 1px solid rgba(50, 50, 50, .1);
      margin-bottom: 10px;
      label {
        text-align: right;
        margin-right: 20px;
        color: #444 !important;
        font-weight: bold !important;
      }
      input {
        border: none;
        background: transparent;
        width: 100%;
        text-align: left;
        padding-left: 0;
        color: #444;
        font-weight: 400;
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
