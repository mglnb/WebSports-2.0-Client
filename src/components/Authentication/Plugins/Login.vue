<script>
import axios from 'axios'

export default {
  props: {
    labels: {
      type: Object,
      default () {
        return {
          title: 'Login',
          user: 'User',
          password: 'Password',
          button: 'Login'
        }
      }
    },
    apiUrl: {
      type: String,
      required: true
    },
    loginRoute: {
      type: String,
      default: 'oauth/token'
    },
    userRoute: {
      type: String,
      default: 'api/user'
    },
    clientId: {
      type: [Number, String],
      default: 2
    },
    secret: {
      type: String,
      required: true
    }
  },
  data () {
    return {
      login: {
        user: '',
        password: ''
      }
    }
  },
  computed: {
    loginUrl () {
      return `${this.apiUrl}/${this.loginRoute}`
    },
    userUrl () {
      return `${this.apiUrl}/${this.userRoute}`
    }
  },
  methods: {
    getHeaders (token) {
      return {
        'Accept': 'application/json',
        'Authorization': `Bearer ${token}`
      }
    },
    handleLogin () {
      this.$Progress.start()

      const postData = {
        grant_type: 'password',
        client_id: this.clientId,
        client_secret: this.secret,
        username: this.login.user,
        password: this.login.password,
        scope: '*'
      }

      this.$http.post(this.loginUrl, postData)
        .then(res=> {
          localStorage['token'] = JSON.stringify(res.body)
          localStorage['user'] = JSON.stringify({username: this.login.user})
          let authUser = {
            user: this.login.user,
            token: res.data.access_token,
            refresh_token: res.data.refresh_token
          }
          let header = this.getHeaders(authUser.token)
          this.$emit('success', {authUser, header})
        }).catch((err)=>{
          this.$emit('error', err)
        })
    }
  }
}
</script>

<template>
  <section class="login">
    <!--header-->

    <!--//header-->
    <!--main-->
    <div class="main-content-agile">
      <div class="sub-main-w3">
        <h2>Login</h2>
        <form @submit.prevent="handleLogin" autocomplete="off">
          <div class="icon1">
            <input class="input" type="email" placeholder="E-mail" v-model="login.user">
          </div>

          <div class="icon2">
            <input class="input" type="password" placeholder="Senha" v-model="login.password" >
          </div>
          <div class="clear"></div>
          <input type="submit" class="button is-primary" v-text="labels.button">
        </form>
      </div>
    </div>
    <!--//main-->
    <!--footer-->

    <!--//footer-->
  </section>
</template>

<style lang="scss">
body {
  background: url(../../../../static/img/bg-login.jpg)no-repeat 0px 0px;
  background-size: cover;
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  -ms-background-size: cover;
  background-attachment: fixed;
  background-position: center;
  background-size: cover;
  font-family: 'Roboto', sans-serif;
  text-align: center;
}

section.login {
  display: flex;
  height: 100vh;
  justify-content: center;
  align-items: center;
}


/*--header--*/

.header-w3l {
  padding: 6em 0 4.5em;
}

.header-w3l h1 {
  font-size: 4.5em;
  color: #ffffff;
  text-shadow: 1px 1px 10px #9a8b89;
  letter-spacing: 4px;
  text-transform: capitalize;
  font-family: 'Raleway', sans-serif;
  word-spacing: 7px;
  letter-spacing: 2px;
  text-align: center
}



/*--//header--*/


/*--main--*/

.main-content-agile {
  margin: 0 auto;
  background: rgba(255, 255, 255, 0.08);
  background: rgba(19, 17, 17, 0.62);
  width: 30%;
  box-shadow: 2px 2px 21px rgba(0, 0, 0, 0.29);
  position: relative;
  padding: 3em;
}

.sub-main-w3 h2 {
  font-size: 2em;
  color: #fff;
  margin-bottom: 1em;
  letter-spacing: 1px;
  font-weight: 400;
  text-transform: capitalize;
  text-shadow: 1px 1px 1px #000;
}

.sub-main-w3 input[type="email"],
.sub-main-w3 input[type="password"] {
  outline: none;
  font-size: 14px;
  border: none;
  box-shadow: 2px 2px 21px rgba(0, 0, 0, 0.29);
  background: rgba(255, 255, 255, 0.21);
  width: 100%;
  color: #fff;
  padding: 10px;
  letter-spacing: 1px;
  font-family: 'Roboto', sans-serif;
}

.sub-main-w3 input[type="submit"] {
  color: #fff;
  background: rgba(54, 14, 120, 0.9);
  border: none;
  width: 100%;
  padding: .8em 0em;
  outline: none;
  font-size: 1.1em;
  cursor: pointer;
  letter-spacing: 1px;
  text-transform: capitalize;
  transition: 0.5s all;
  -webkit-transition: 0.5s all;
  -o-transition: 0.5s all;
  -moz-transition: 0.5s all;
  -ms-transition: 0.5s all;
  font-family: 'Roboto', sans-serif;
  margin-top: 1.5em;
}

.sub-main-w3 input[type="submit"]:hover {
  background: rgba(54, 14, 120, 1);
  color: #fff;
}

.icon1,
.icon2 {
  position: relative;
  margin-top: 1.3em;
}

.icon1 {
  padding-bottom: 1em!important;
}

.rem-w3 {
  margin: 1.5em 0;
}

span.check-w3 {
  float: left;
  color: #fff;
  font-size: 13.5px;
  letter-spacing: 1px;
}

.rem-w3 a {
  color: #fff;
  float: right;
  font-size: 13.5px;
  letter-spacing: 1px;
}

.rem-w3 a:hover {
  color: #dd4b39;
  transition: 0.5s all;
  -webkit-transition: 0.5s all;
  -o-transition: 0.5s all;
  -moz-transition: 0.5s all;
  -ms-transition: 0.5s all;
}

.sub-main-w3 h6 {
  color: #fff;
  font-size: 15px;
  letter-spacing: 1px;
  text-transform: capitalize;
  margin: 2em 0 1em;
  text-shadow: 1px 1px 2px #000000;
}



/*-- social-icons --*/

.navbar-right.social-icons a {
  display: inline-block;
  padding: 10px 0px;
  text-decoration: none;
  width: 40%;
  float: left;
  margin: 0 4.7%;
  letter-spacing: 1px;
  font-weight: 400;
  cursor: pointer;
  color: #fff;
  text-shadow: 1px 1px 1px #000;
  font-size: 14px!important;
  border: 1px solid #fff;
  transition: 0.5s all;
  -webkit-transition: 0.5s all;
  -o-transition: 0.5s all;
  -moz-transition: 0.5s all;
  -ms-transition: 0.5s all;
}

.navbar-right.social-icons a.one-w3ls:hover {
  background: #3b5998;
  float: left;
}

.navbar-right.social-icons a.two-w3ls:hover {
  background: #dd4b39;
}

.navbar-right.social-icons p i {
  color: #fff;
  font-size: 24px;
}

.navbar-right.social-icons a i {
  padding-right: 15px;
}



/*-- //social-icons --*/


/*-- checkbox --*/

.anim {
  position: relative;
  display: block;
  margin: 1.5em 0;
}

label.anim span,
label.anim a {
  color: #fff;
  font-size: 13px;
  display: inline;
  letter-spacing: 1px;
  text-transform: capitalize;
  float: left;
}

label.anim a {
  display: inline-block;
  text-decoration: none;
  float: right;
}

.wthree-text ul li {
  displaY: block;
}

.wthree-text ul li:nth-child(1) {
  margin-right: 36px;
}

.agileits-login label {
  font-size: 1em;
  color: #fff;
  font-weight: 400;
  cursor: pointer;
  position: relative;
}

input.checkbox {
  background: transparent;
  cursor: pointer;
  width: 1.2em;
  height: 0;
  margin: 0 6px 0 0!important;
  float: left;
  &:before {
    content: "";
    position: absolute;
    width: 1.2em;
    height: 1.2em;
    background: inherit;
    cursor: pointer;
  }
  &:after {
    content: "";
    transition: .4s ease-in-out;
    position: absolute;
    top: 0px;
    left: 0px;
    z-index: 1;
    width: 1.1em;
    height: 1.1em;
    /* margin-top: 4px; */
    border: 1px solid #ffffff;
  }
  & ~ span {
    padding-left: 22px;
    cursor: pointer;
  }
}


input.checkbox:checked:after {
  transform: rotate(-45deg);
  height: .5rem;
  border-color: #fff;
  border-top-color: transparent;
  border-right-color: transparent;
}

.anim input.checkbox:checked:after {
  transform: rotate(-45deg);
  height: .5rem;
  border-color: transparent;
  border-right-color: transparent;
  animation: .4s rippling .4s ease;
  animation-fill-mode: forwards;
  top: 5px;
}

@keyframes rippling {
  50% {
    border-left-color: #d24747;
  }
  100% {
    border-bottom-color: #fff;
    border-left-color: #fff;
  }
}



/*-- //checkbox --*/


/*--//main--*/


/*--footer--*/

.footer {
  padding: 10em 0 14.7em;
}

.footer p {
  font-size: 15px;
  color: #fff;
  font-weight: 500;
  letter-spacing: 2px;
}

.footer p a {
  color: #ff4c4c;
}

.footer p a:hover {
  text-decoration: underline;
}



/*--//footer--*/


/*--placeholder-color--*/

 ::-webkit-input-placeholder {
  color: #fff;
}

 :-moz-placeholder {
  /* Firefox 18- */
  color: #fff;
}

 ::-moz-placeholder {
  /* Firefox 19+ */
  color: #fff;
}

 :-ms-input-placeholder {
  color: #fff;
}



/*--//placeholder-color--*/


/*--responsive--*/

@media(max-width: 1680px) {
  .header-w3l h1 {
    font-size: 4em;
  }
  .header-w3l {
    padding: 7em 0 3.5em;
  }
  .footer {
    padding: 8em 0;
  }
}

@media(max-width: 1600px) {
  .header-w3l {
    padding: 4.5em 0 3.5em;
  }
  .header-w3l h1 {
    font-size: 3.5em;
  }
  .footer {
    padding: 3.07em 0;
  }
}

@media(max-width: 1440px) {
  .header-w3l h1 {
    font-size: 2.8em;
  }
  .wthree-pro h2 {
    font-size: 1.5em;
  }
  .header-w3l {
    padding: 2.5em 0;
  }
  .main-content-agile {
    width: 38%;
  }
  .footer {
    padding: 3em 0;
  }
  .header-w3l {
    padding: 4.5em 0;
  }
  .footer {
    padding: 5.37em 0;
  }
}

@media(max-width: 1366px) {
  .main-content-agile {
    width: 40%;
  }
  .header-w3l {
    padding: 3em 0;
  }
  .footer {
    padding: 2.74em 0;
  }
}

@media(max-width: 1280px) {
  .main-content-agile {
    width: 43%;
  }
  .header-w3l {
    padding: 3.5em 0;
  }
  .footer {
    padding: 3.24em 0;
  }
}

@media(max-width: 1080px) {
  .header-w3l h1 {
    font-size: 2.6em;
  }
  .wthree-pro h2 {
    font-size: 1.4em;
    padding: 0.8em 0 0.8em 1.8em;
  }
  .main-content-agile {
    width: 49%;
  }
  .sub-main-w3 {
    padding: 2em;
  }
  .sub-main-w3 input[type="submit"] {
    padding: .7em 4em;
    font-size: 1em;
  }
}

@media(max-width: 1024px) {
  .main-content-agile {
    width: 51%;
  }
  .header-w3l {
    padding: 5em 0 3em;
  }
  .footer {
    padding: 3.44em 0;
  }
  .sub-main-w3 input[type="email"],
  .sub-main-w3 input[type="password"] {
    padding: 9.74px;
  }
}

@media(max-width: 991px) {
  .header-w3l h1 {
    font-size: 2.5em;
  }
}

@media(max-width: 900px) {
  .main-content-agile {
    width: 59%;
  }
}

@media(max-width: 800px) {
  .header-w3l h1 {
    font-size: 2.3em;
  }
  .main-content-agile {
    width: 61%;
  }
}

@media(max-width: 768px) {
  .header-w3l {
    padding: 3em 0 2em;
  }
  .main-content-agile {
    width: 65%;
    padding: 2em;
  }
  .footer {
    padding: 3em 0 2em 0;
  }
}

@media(max-width:667px) {
  .footer p {
    font-size: 14px;
    letter-spacing: 1px;
    padding: 0 3px;
  }
}

@media(max-width:600px) {
  .main-content-agile {
    width: 73%;
    padding: 1em;
  }
}

@media(max-width:568px) {
  .footer p {
    padding: 0 5px;
    line-height: 1.9em;
  }
}

@media(max-width:480px) {
  .header-w3l h1 {
    font-size: 2em;
  }
  .sub-main-w3 {
    padding: 1em;
  }
  .navbar-right.social-icons a {
    margin: 0 3.7%;
  }
  .sub-main-w3 h2 {
    font-size: 1.6em;
  }
}

@media(max-width:414px) {
  .main-content-agile {
    width: 87%;
    padding: 1em;
  }
  .header-w3l {
    padding: 2em 0 1.2em;
  }
  .header-w3l h1 {
    font-size: 1.8em;
    line-height: 1.5em;
  }
  .footer {
    padding: 2em 0 1em 0;
  }
  .sub-main-w3 h2 {
    font-size: 1.5em;
  }
  .sub-main-w3 h6 {
    margin: 1.3em 0 1em;
  }
}

@media(max-width:375px) {
  .icon1,
  .icon2 {
    position: relative;
    margin-top: 0.5em;
  }
  .navbar-right.social-icons a i {
    padding-right: 5px;
  }
}

@media(max-width: 320px) {
  .header-w3l h1 {
    font-size: 1.6em;
    line-height: 1.5em;
  }
  .sub-main-w3 {
    padding: 0em;
  }
  .header-w3l {
    padding: 1em 0 0.5em;
  }
}



/*--//responsive--*/
</style>
