<template>
<div class="row">

  <div class="col-md-12">
    <div class="card card-plain">
      <paper-table @create="cliente_class = 'cliente--active'" @update="handleUpdate" type="hover" :title="table.title" :sub-title="table.subTitle" :data="clientes" :columns="table.columns">

      </paper-table>
      <div class="cliente" :class="cliente_class">
        <div class="cliente__header">
          <div class="cliente--column">
            <span class="cliente__title">Edição de cliente</span>
            <span class="cliente__subtitle">Dê dois cliques no campo para editar e enter para salvar</span>
          </div>
          <span class="ti-close cliente__close" @click.stop="cliente_class = 'cliente--closed'"></span>
        </div>
        <div class="cliente__body">
          <div class="cliente__input-group">
            <label for="">NOME</label>
            <input type="text"  v-model="cliente.nome">
          </div>
          <div class="cliente__input-group">
            <label for="">E-MAIL</label>
            <input type="text"  v-model="cliente.email">
          </div>
          <div class="cliente__input-group">
            <label for="">CPF</label>
            <input type="text" v-model="cliente.cpf" v-mask="['###.###.###-##']">
          </div>
          <div class="cliente__input-group">
            <label for="">SALDO</label>
            <input type="text" v-model="cliente.saldo" v-mask="['R$ ###,00']">
          </div>
          <div class="cliente__input-group">
            <label for="">CEP</label>
            <input type="text" @blur="findCep" v-mask="['#####-###']" v-model="cliente.cep">
          </div>
          <div class="cliente__input-group">
            <label for="">Endereço</label>
            <div class="flex flex-row">
              <input type="text" placeholder="Rua" v-model="cliente.endereco.rua">
              <input type="text" placeholder="Numero" v-model="cliente.endereco.numero">
              <input type="text" placeholder="Complemento" v-model="cliente.endereco.complemento">
            </div>
          </div>
           <div class="">
            <button @click="handleCreate($event)">ENVIAR</button>
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
        cep: '',
        endereco: {
          rua: '',
          numero: '',
          complemento: ''
        },
        cpf: '',
        saldo: '',
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
    handleUpdate(payload) {
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

      this.$http.put('//websports.herokuapp.com/api/clientes/' + payload.data.id, data).then((response) => {
      })
      this.$store.dispatch("load-clientes");
    },
    handleCreate(payload) {
      console.log(payload)
      this.$http.post('//websports.herokuapp.com/api/clientes', this.cliente)
      this.$store.dispatch("load-clientes");
    },
    findCep() {
      axios.get(`http://viacep.com.br/ws/${this.cliente.cep}/json/`)
        .then(response => {
          this.cliente.endereco = {
            rua : response.data.logradouro,
            complemento : response.data.complemento,
            numero : ''
          }
        })
    }
  },
  mounted() {
  }
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
      
      input:first-child {
        width: 50%
      }
      input:nth-child(2) {
        margin: 0 10px;
        width: 25%;
      }
      input:last-child {
        width: 25%;
      }
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
