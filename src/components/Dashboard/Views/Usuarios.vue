<template>
<div class="row">

  <div class="col-md-12">
    <div class="card card-plain">
      <paper-table @view="handleView" @delete="handleDelete" @create="handleCreate" @update="handleUpdate" type="hover" :title="table.title" :sub-title="table.subTitle" :data="users" :columns="table.columns">

      </paper-table>
      <div class="usuario" :class="usuario_class">
        <div class="usuario__header">
          <div class="usuario--column">
            <span class="usuario__title">Criação de usuario</span>
          </div>
          <span class="ti-close usuario__close" @click.stop="resetFields($event)"></span>
        </div>
        <div class="usuario__body">
          <form action="">
            <div class="usuario__input-group">
              <label>NOME DO USUARIO</label>
              <input type="text" v-model="usuario.name" required>
            </div>
            <div class="usuario__input-group">
              <label>E-MAIL</label>
              <input type="email" v-model="usuario.email" required>
            </div>
            <div class="usuario__input-group">
              <label>SENHA</label>
              <input type="password" v-model="usuario.password" required>
            </div>
            <div class="usuario__button-group">
              <button @click.prevent="handleCreate($event)">Enviar</button>
              <button @click.prevent="resetFields($event)">Limpar</button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>

</div>
</template>
<script>
import PaperTable from "components/UIComponents/PaperTable.vue";

const tableColumns = [{
    name: "Nome do Usuario"
  },
  {
    name: "Email"
  },
  {
    name: "Senha"
  }
];

export default {
  created() {
    // this.$Progress.start()
    this.$store.dispatch("load-users");
  },
  components: {
    PaperTable
  },
  computed: {
    users() {
      return this.$store.state.users;
    }
  },
  mounted() {
    console.log(this.users);

  },
  methods: {
    async handleUpdate(payload) {
      this.$Progress.start()
      payload.data.column = payload.data.column.replace('Nome do Usuario', 'name');
      payload.data.column = payload.data.column.replace('Id', 'id');
      payload.data.column = payload.data.column.replace('Email', 'email');
      payload.data.column = payload.data.column.replace('Senha', 'password');
      console.log(payload.data);
      let data = {}
      data[payload.data.column] = payload.data.value
      await this.$http.put(`//websports.herokuapp.com/api/users/${payload.data.id}`, data)
        .then(response => {
          this.$Progress.finish()
        })
        .catch(err => {
          this.$Progress.fail()
        })
      payload.e.target.setAttribute('readonly', 'true')
      this.$store.dispatch("load-users");
    },
    async handleCreate(payload) {
      if (payload.target.localName == "span") {
        this.usuario_class = "usuario--active"
        return
      }
      if(this.usuario.name == "" || this.usuario.email == "" || this.usuario.password == "") {
        return
      }
      this.$Progress.start()
      await this.$http.post('//websports.herokuapp.com/api/users', this.usuario)
        .then(res => {
          this.$Progress.finish()
        })
        .catch(err => {
          this.$Progress.fail()
        })
        this.resetFields();
      this.$store.dispatch('load-users')
    },
    async handleDelete(payload) {
      this.$Progress.start();
      await this.$http.delete(`//websports.herokuapp.com/api/users/${payload}`)
        .then(res => {
          this.$Progress.finish()
        })
        .catch(err => {
          this.$Progress.fail()
        })
      this.$store.dispatch('load-users')

    },
    async handleView(payload) {
      this.$Progress.start()
      await this.$http.get(`//websports.herokuapp.com/api/users/${payload}`)
                    .then(res => {
                      this.usuario = {
                        name: res.body.name,
                        email: res.body.email,
                        password: '*********'
                      }
                      this.$Progress.finish()
                    })
                    .catch(err => {
                      this.$Progress.fail()
                    })
      this.usuario_class = "usuario--active"      
    },
    resetFields(e) {
      if (e.target.localName == "span") {
        this.usuario_class = "reserva--closed"
      }
      this.usuario = {
        name: '',
        email: '',
        password: '',
      }
    }
  },
  data() {
    return {
      usuario_class: '',
      usuario: {
        name: '',
        email: '',
        password: '',
      },
      sort: "Id",
      table: {
        title: "Listagem de Usuários",
        subTitle: "Para qualquer alteração, clique duas vezes em cima do registro",
        columns: [tableColumns]
      }
    };
  }
};
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

.usuario {
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
