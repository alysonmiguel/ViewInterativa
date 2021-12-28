
<script>
import qs from "qs";
import Cookie from "js-cookie";
export default {
  name: "Login",
  data() {
    return {
      email: "gerente_sgeol_test@test.com",
      password: "1234",
    };
  },
  methods: {
    async tokenApp() {
      /**
       *  Recuperando o token da aplicação
       */
       const response  = await (fetch(`http://localhost:8080/sgeol-dm/security/auth/login/app`, {
          method: "POST",
          headers: {
            "Content-Type": "application/x-www-form-urlencoded",
            "Access-Control-Allow-Origin": "*",
            Accept: "*/*",
          },
          body: qs.stringify({
            username: "app_sgeol_test@test.com",
            password: "1234",
          }),
        }
      ))
      const tokens = await response.json()
      Cookie.set("app_access_token", tokens.access_token);
      Cookie.set("app_expires_in", tokens.expires_in);
      Cookie.set("app_refresh_token", tokens.refresh_token);
    },
    async tokenUser() { 
       /**
       * Recupera um token de um usuário vinculado a uma aplicação
       */
      let form = Object.assign(
        {},
        {
          username: this.email,
          password: this.password,
          login: "",
          "application-token": Cookie.get("app_access_token"),
          "app-token": Cookie.get("app_access_token"),
        }
      );
     
      const response = await (fetch(
        `http://localhost:8080/sgeol-dm/security/auth/login/user`,
        {
          method: "POST",
          headers: {
            "Content-Type": "application/x-www-form-urlencoded",
            "Access-Control-Allow-Origin": "*",
            Accept: "*/*",
            Authorization:
              "ZTJiYjc5NDEtYTA1Zi00ZWI3LThkNDItOTMwZjY3OGFlZmFhOmE0YTQ3NWM5LTU3NGQtNDBiYy1hZDM1LTU4ZDUwOTNiOThkNK==",
          },
          body: qs.stringify(form),
        }
      ))
      const tokens = await response.json()
      if (tokens.access_token) {
        Cookie.set("user_access_token", tokens.access_token);
        Cookie.set("user_expires_in", tokens.expires_in);
        Cookie.set("user_refresh_token", tokens.refresh_token);
      }
      this.$router.push({ name: "Home" });
    },
  },
  mounted(){
    this.tokenApp()
    this.tokenUser()
  }
};
</script>
