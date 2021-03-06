---
layout: post
title: "๐พ Javascript Adapter๋?"
date: 2021-08-28 16:40:00 +0900
categories: Keycloak
---

## Javascript Adapter

์ด ๋ผ์ด๋ธ๋ฌ๋ฆฌ๋ keycloak.js์ keycloak์๋ฒ์์ ๋ฐ๋ก ๋ถ๋ฆด ์๋ ์๊ณ , zip ์์นด์ด๋ธํ์ผ๋ก๋ ์กด์ฌํ๋ค.

๊ฐ์ด๋์ ๋ฐ๋ฅด๋ฉด keycloak์๋ฒ์์ javascript adapter๋ฅผ ๋ถ๋ฅด๋๊ฒ ๊ฐ์ฅ ์ข๋ค๊ณ  ํ๋ค. ๊ทธ๋ ๊ฒ ํ๋ฉฐ ์๋ฒ๋ฅผ ์๊ทธ๋ ์ด๋ ์ํฌ ๋ ์๋์ผ๋ก ๊ฐ์ด ์๋ฐ์ดํธ ๋๊ธฐ ๋๋ฌธ์ด๋ค. ๋ง์ฝ adapter๋ฅผ ์ดํ๋ฆฌ์ผ์ด์์ผ๋ก copyํด์ ์ฌ์ฉํ๋ค๋ฉฐ ์๋ฒ๊ฐ ์๊ทธ๋ ์ด๋ ๋  ๋ adapter๋ ์๋์ผ๋ก ์๊ทธ๋ ์ด๋ ์์ผ์ค์ผ ํ๋ค.

client-side ์ดํ๋ฆฌ์ผ์ด์์ ์ฌ์ฉํ  ๋ ์์งํด์ผ๋๋ ๋ถ๋ถ์ client๋ public client ์ด์ด์ผ ํ๊ธฐ ๋๋ฌธ์ client-side ์ดํ๋ฆฌ์ผ์ด์์ client credential์ ์์ ํ๊ฒ ์ ์ฅํด๋์ ๋ฐฉ๋ฒ์ด ์๋ค๋ ๊ฒ์ด๋ค. ์ด ๋ถ๋ถ์ client๋ฅผ ์ํด ์ง์ ํด ๋์ redirect URI๋ค์ด ์ ํํ๊ณ  ๋ชํ(specific)ํด์ผ ํ๋ค๋ ๊ฒ์ ์ค์ํ๊ฒ ๋ง๋ ๋ค.

### JavaScript adapter๋ฅผ ์ฌ์ฉํ๊ธฐ ์ํด์

1. Keycloak Administration Console์์ client๋ฅผ ์์ฑํ๋ค. Access Type์ด public์ผ๋ก ์ค์ ๋ผ์๋์ง ์ฒดํฌํด์ผ ๋๋ค.
2. Valid Redirect URIs์ Web Origins๋ฅผ ์ค์ ํด์ค๋ค. ๋ณด์์ ์ผ๋ก ์ทจ์ฝํด์ง์ง ์๊ฒ ๊ฐ๋ฅํ ์์ธํ ๋ด์ฉ๋ค์ ๋ด์์ผ ํ๋ค.
3. client๊ฐ ์์ฑ๋๋ค๋ฉด Installation ํญ์์ Format Option์ผ๋ก Keycloak OIDC JSON์ ์ ํํ๊ณ  Download๋ฅผ ํด๋ฆญํ๋ค. ๋ค์ด๋ก๋ ๋ keycloak.jsonํ์ผ์ ๋น์ ์ HTMLํ์ด์ง์ ๊ฐ์ ์์น์ ๋น ์  ์น์๋ฒ์์ hosted๋ผ์ผ ๋๋ค. 
    - ๋์ฒด ๋ฐฉ์์ผ๋ก, configuration๊ณผ์ ์ ๋์ด๊ฐ๊ณ  adpater์ ๋ํ ์ค์ ์ ์๋์ ์ผ๋ก ํ  ์๋ ์๋ค.
    - JavaScript adapter๋ฅผ initialize ํด์ฃผ๋ ์ฝ๋ ์์์ด๋ค.

    ```html
    <head>
        <script src="keycloak.js"></script>
        <script>
            var keycloak = new Keycloak();
            keycloak.init().then(function(authenticated) {
                alert(authenticated ? 'authenticated' : 'not authenticated');
            }).catch(function() {
                alert('failed to initialize');
            });
        </script>
    </head>
    ```

    - ๋ง์ฝ keycloak.json ํ์ผ์ด ๋ค๋ฅธ ์์น์ ์๋ค๋ฉด ์๋์ ๊ฐ์ด ๋ช์ํด์ฃผ๋ฉด ๋๋ค.

    ```jsx
    var keycloak = new Keycloak('http://localhost:8080/myapp/keycloak.json');
    ```

    - keycloak ๊ฐ์ฒด๋ฅผ ์์ฑํ  ๋ configuration์ ์ค์ ํด์ค ์๋ ์๋ค.

    ```jsx
    var keycloak = new Keycloak({
        url: 'http://keycloak-server/auth',
        realm: 'myrealm',
        clientId: 'myapp'
    });
    ```

๊ธฐ๋ณธ์ ์ผ๋ก authentication์ ํ๊ธฐ ์ํด์  loginํจ์๋ฅผ ๋ถ๋ฌ์ผ ๋๋๋ฐ, adapter๊ฐ ์๋์ผ๋ก authentication ๊ณผ์ ์ ๊ฑฐ์น๊ฒ ํ๋ ๋ ๊ฐ์ง ์ต์์ด ์๋ค.

- login-required : ์ฌ์ฉ์๊ฐ Keycloak์ ๋ก๊ทธ์ธ ๋ผ์๋์ง ํ์ธํ๊ฑฐ๋ ๋ก๊ทธ์ธ ํ์ด์ง๊ฐ ์๋ ์๋ค๋ฉด ํ์ด์ง๋ฅผ ๋์์ค๋ค.
- check-sso : ์ฌ์ฉ์๊ฐ Keycloak์ ๋ก๊ทธ์ธ ๋ผ์๋์ง๋ง ํ์ธํ๊ณ , ์๋ผ์์ผ๋ฉด ์๋ผ์๋ ์ํ๋ก ๊ทธ๋ฅ ์ดํ๋ฆฌ์ผ์ด์ํ๋ฉด์ผ๋ก ๋ค์ ๋์๊ฐ๋ค.

## Implicit and Hybrid Flow

default๋ก, JavaScript adapter๋ Authorization Code ํ๋ก์ฐ๋ฅผ ์ฌ์ฉํ๋ค.

์ด ํ๋ก์ฐ๋ฅผ ํตํด Keycloak ์๋ฒ๋ authentication code๊ฐ ์๋ authorization code๋ฅผ ์ดํ๋ฆฌ์ผ์ด์์ ๋ฐํํ๋ค. JavaScript adapter๋ ๋ธ๋ผ์ฐ์ ๊ฐ ์ดํ๋ฆฌ์ผ์ด์์ผ๋ก redirected ๋ ํ์ code๋ฅผ access token์ด๋ refresh token์ผ๋ก ๊ตํํ๋ค.

Keycloak์ ์ธ์ฆ๊ณผ์  ์ฑ๊ณต ํ access token์ด ๋ฐ๋ก ๋ณด๋ด์ง๋ implicit flow๋ ์ง์ํ๋ค. ์ด๊ฑด ํ ํฐ์ ์ํ ์ถ๊ฐ์ ์ธ ์ฝ๋ ๊ตํ์ด ์ด๋ฃจ์ด์ง์ง ์๊ธฐ ๋๋ฌธ์ ๊ธฐ์กด ํ๋ก์ฐ๋ณด๋ค ๋ ๋์ ์ฑ๋ฅ์ ๋ณด์ฌ์ค ์ ์์ง๋ง, access token์ด ๋ง๋ฃ๋์ ๋ ์ํฅ์ ์ค ์ ์๋ค.

access token์ url์ ํฌํจ์์ผ ๋ณด๋ด๋ ๊ฒ์ ๋ณด์์ ์ทจ์ฝํ  ์ ์๋ค. ์๋ฅผ ๋ค์ด ์น ์๋ฒ ๋ก๊ทธ๋ ๋ธ๋ผ์ฐ์  ํ์คํ ๋ฆฌ๋ฅผ ํตํด ํ ํฐ์ด ๋ธ์ถ ๋  ์ ์๋ค.

implicit ํ๋ก์ฐ๋ฅผ ํ์ฑํ ์ํค๋ ค๋ฉด, Keycloak Administration Console์์ Implicit Flow Enabled ํ๋๊ทธ๋ฅผ ํ์ฑํ์ํค๋ฉด ๋๋ค. ์๋๋ฉด init ์์ ์์ flowํ๋ผ๋ฏธํฐ๋ฅผ implicit์ผ๋ก ์ค์ ํ๋ฉด ๋๋ค.

```jsx
keycloak.init({
    flow: 'implicit'
})
```

ํ๋ ์ฃผ์ํด์ผ ํ  ์ ์ access token๋ง ์ ๊ณต๋๊ณ  refresh token์ ์ ๊ณต๋์ง ์๋๋ค๋ ์ ์ด๋ค. ์ด๊ฒ์ access token์ด ๋ง๋ฃ๋๋ฉด ๋ค์ Keycloak์ผ๋ก redirect ํด์ ์๋ก์ด access token์ ๋ฐ์์ผ ๋๋ค๋ ๊ฒ์ ์๋ฏธํ๋ค.

Keycloak์ Hybrid flow๋ ์ง์ํ๋ค.

์ฌ์ฉ์๋ Standard Flow Enabled์ Implicit Flow Enabled ํ๋๊ทธ ๋ชจ๋๋ฅผ admin console์์ ํ์ฑํ ์์ผ์ผ ๋๋ค. ๊ทธ๋ฌ๋ฉด Keycloak ์๋ฒ๋ code์ token ๋ชจ๋๋ฅผ ์ดํ๋ฆฌ์ผ์ด์์ ๋ณด๋ธ๋ค. access token์ ๋ฐ๋ก ์ฌ์ฉ๋  ์ ์๊ณ , code๋ access token์ด๋ refresh token์ผ๋ก ๊ตํ๋  ์ ์๋ค. implicit ํ๋ก์ฐ์ ๋น์ทํ๊ฒ, hybird ํ๋ก์ฐ๋ acceess token์ ๋ฐ๋ก ๋ฐ์ ์ธ ์ ์๊ธฐ ๋๋ฌธ์ ์ฑ๋ฅ๋ฉด์์ ์ข๋ค. ํ์ง๋ง ํ ํฐ์ด url ์์ ํฌํจ๋ผ์ ๋ณด๋ด์ง๊ธฐ ๋๋ฌธ์ ๋ณด์์ ์ทจ์ฝํ  ์ ์๋ค.

Hybird ํ๋ก์ฐ์ ํ๊ฐ์ง ์ฅ์ ์ refresh token์ ๋ง๋ค ์ ์๋ค๋ ๊ฒ์ด๋ค. init์์ ์์ flowํ๋ผ๋ฏธํฐ๋ฅผ hybrid๋ก ์ค์ ํ๋ฉด ๋๋ค.

## Modern Browsers with Tracking Protection

๋ช๋ช ๋ธ๋ผ์ฐ์ ์ ์ต์  ๋ฒ์ ์์  ํฌ๋กฌ์ SamSite ๊ฐ์ thirdํํฐ๋ก๋ถํฐ ์ฌ์ฉ์๊ฐ ์ถ์ ๋นํ๋ ๊ฒ์ ๋ฐฉ์งํ๊ฑฐ๋, thirdํํฐ ์ฟ ํค๋ฅผ ์์ ํ ์ฐจ๋จ์ํค๊ธฐ ์ํด ๋ค์ํ ์ฟ ํค ์ ์ฑ๋ค์ด ์ ์ฉ๋์๋ค. ์ด๋ฌํ ์ ์ฑ๋ค์ ์์ผ๋ก ๋ ์๊ฒฉํด์ง๊ณ  ๋ ๋ง์ ๋ธ๋ผ์ฐ์ ์ ์ ์ฉ๋  ๊ฒ์ผ๋ก ์์๋๋ค. thirdํํฐ context์ ์ฟ ํค๋ค์ด ์์ ํ ์ง์์ด ์๋๊ฒ๋ ํ๊ฑฐ๋ ๋ธ๋ผ์ฐ์ ์ ์ํด ์ฐจ๋จ๋๋ ๊ฒฐ๊ณผ๋ฅผ ์ผ๊ธฐํ  ์ ์๋ค. ๋ฏธ๋์ adapter ๊ธฐ๋ฅ๋ค์ด ์ด๊ฒ์ ์ํฅ์ ๋ฐ๊ฑฐ๋ deprecated ๋  ์๋ ์๋ค.

### Browsers with Blocked Third-Party Cookies

Js adapter์ ์ํด ๋ธ๋ผ์ฐ์ ์ ํ๋์ด ๊ฐ์ง๋๋ฉด Session Status iframe์ ์ง์๋์ง ์๊ณ  ์๋์ผ๋ก disabled๋๋ค. ์ด๊ฒ์ adapter๊ฐ Single Sign-Out ๊ฐ์ง๋ฅผ ์ํด session cookie๋ฅผ ์ฌ์ฉํ์ง ๋ชปํ๊ณ , ์ ์ ์ผ๋ก token์ ์์กดํด์ผ ๋๋ค๋ ๊ฒ์ ์๋ฏธํ๋ค. ์ฌ์ฉ์๊ฐ ๋ค๋ฅธ ์ฐฝ์์ ๋ก๊ทธ์์์ ํ๋ฉด, JavaScript adapter๋ฅผ ์ฌ์ฉํ๋ ์ดํ๋ฆฌ์ผ์ด์์ access token์ refreshํ๊ธฐ ์ ๊น์ง๋ ๋ก๊ทธ์์ ๋์ง ์๋๋ค. ๊ทธ๋ ๊ธฐ ๋๋ฌธ์, Access token์ ์๋ช์ ๋น๊ต์  ์งง๊ฒ ์ค์ ํด์ ๋ก๊ทธ์์์ ์ข ๋ ๋นจ๋ฆฌ ๊ฐ์งํ๊ฒ๋ ํ๋ ๊ฒ์ ๊ถ์ฅํ๋ค. (์ฐธ๊ณ : [https://www.keycloak.org/docs/latest/server_admin/#_timeouts](https://www.keycloak.org/docs/latest/server_admin/#_timeouts))

slient check-sso๋ ์ง์๋์ง ์์ผ๋ฉด default๋ก๋ regular(non-silent) check-sso๋ก falls back๋๋ค. ์ด ์ฒ๋ฆฌ๋ init๋ฉ์๋์์ ๋์ด๊ฐ๋ silentCheckSsoFallback ํ๋ผ๋ฏธํฐ๋ฅผ false๋ก ํด์ค์ผ๋ก์จ ๋ ์ ์๋ค. ์ด ๊ฒฝ์ฐ์, ๋ง์ฝ ์๊ฒฉํ ๋ธ๋ผ์ฐ์ ์ ํ๋์ด ๊ฐ์ง๋๋ฉด check-sso๋ ์์ ํ disabled๋๋ค.

regular check-sso ๋ํ ์ํฅ์ ๋ฐ๋๋ค. Session Status iframe์ด ์ง์๋์ง ์๊ธฐ ๋๋ฌธ์, adpater๊ฐ ์ฌ์ฉ์์ ๋ก๊ทธ์ธ ์ํ์ ๋ํด ์ฒดํฌํ๋๋ก initialized ๋๋ค๋ฉด ์ถ๊ฐ์ ์ธ Keycloak์ผ๋ก์ redirection์ด ์ด๋ฃจ์ด์ ธ์ผ ํ๋ค. ์ด๊ฒ์ ์ฌ์ฉ์๊ฐ ๋ก๊ทธ์ธ ๋๋์ง ํ์ธํ๊ณ  ๋ก๊ทธ์์ ๋์ ๋๋ง redirectํด์ฃผ๋, iframe์ด ์ฌ์ฉ๋๋ standardํ ์์๊ณผ๋ ์ฐจ์ด๊ฐ ์๋ค. (13.1.์ผ๋ก ์์ํ๋ Safari ๋ฑ์์ ์ํฅ์ ๋ฐ๋๋ค)

## JavaScript Adapter Reference

### **Constructor**

`new Keycloak();`  
`new Keycloak('http://localhost/keycloak.json');`  
`new Keycloak({ url: 'http://localhost/auth', realm: 'myrealm', clientId: 'myApp' });`  

### **Properties**

**authenticated** Isย `true`ย if the user is authenticated,ย `false`ย otherwise.

**token** The base64 encoded token that can be sent in theย `Authorization`ย header in requests to services.

**tokenParsed** The parsed token as a JavaScript object.

**subject** The user id.

**idToken** The base64 encoded ID token.

**idTokenParsed** The parsed id token as a JavaScript object.

**realmAccess** The realm roles associated with the token.

**resourceAccess** The resource roles associated with the token.

**refreshToken** The base64 encoded refresh token that can be used to retrieve a new token.

**refreshTokenParsed** The parsed refresh token as a JavaScript object.

**timeSkew** The estimated time difference between the browser time and the Keycloak server in seconds. This value is just an estimation, but is accurate enough when determining if a token is expired or not.

**responseMode** Response mode passed in init (default value is fragment).

**flow** Flow passed in init.

method์ options์ ๋ํ reference๋ ์๋ณธ ์ฌ์ดํธ ์ฐธ๊ณ 

[Securing Applications and Services Guide](https://www.keycloak.org/docs/latest/securing_apps/#javascript-adapter-reference)