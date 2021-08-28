---
layout: post
title: "🎾 Javascript Adapter란?"
date: 2021-08-28 16:40:00 +0900
categories: Keycloak
---

## Javascript Adapter

이 라이브러리는 keycloak.js의 keycloak서버에서 바로 불릴 수도 있고, zip 아카이브파일로도 존재한다.

가이드에 따르면 keycloak서버에서 javascript adapter를 부르는게 가장 좋다고 한다. 그렇게 하며 서버를 업그레이드 시킬 때 자동으로 같이 업데이트 되기 때문이다. 만약 adapter를 어플리케이션으로 copy해서 사용한다며 서버가 업그레이드 될 때 adapter도 수동으로 업그레이드 시켜줘야 한다.

client-side 어플리케이션을 사용할 때 숙지해야되는 부분은 client는 public client 이어야 하기 때문에 client-side 어플리케이션엔 client credential을 안전하게 저장해놓을 방법이 없다는 것이다. 이 부분은 client를 위해 지정해 놓은 redirect URI들이 정확하고 명확(specific)해야 한다는 것을 중요하게 만든다.

### JavaScript adapter를 사용하기 위해서

1. Keycloak Administration Console에서 client를 생성한다. Access Type이 public으로 설정돼있는지 체크해야 된다.
2. Valid Redirect URIs와 Web Origins를 설정해준다. 보안적으로 취약해지지 않게 가능한 상세한 내용들을 담아야 한다.
3. client가 생성됐다면 Installation 탭에서 Format Option으로 Keycloak OIDC JSON을 선택하고 Download를 클릭한다. 다운로드 된 keycloak.json파일은 당신의 HTML페이지와 같은 위치의 당 신 웹서버에서 hosted돼야 된다. 
    - 대체 방안으로, configuration과정을 넘어가고 adpater에 대한 설정을 수동적으로 할 수도 있다.
    - JavaScript adapter를 initialize 해주는 코드 예시이다.

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

    - 만약 keycloak.json 파일이 다른 위치에 있다면 아래와 같이 명시해주면 된다.

    ```jsx
    var keycloak = new Keycloak('http://localhost:8080/myapp/keycloak.json');
    ```

    - keycloak 객체를 생성할 때 configuration을 설정해줄 수도 있다.

    ```jsx
    var keycloak = new Keycloak({
        url: 'http://keycloak-server/auth',
        realm: 'myrealm',
        clientId: 'myapp'
    });
    ```

기본적으로 authentication을 하기 위해선 login함수를 불러야 되는데, adapter가 자동으로 authentication 과정을 거치게 하는 두 가지 옵션이 있다.

- login-required : 사용자가 Keycloak에 로그인 돼있는지 확인하거나 로그인 페이지가 안떠있다면 페이지를 띄워준다.
- check-sso : 사용자가 Keycloak에 로그인 돼있는지만 확인하고, 안돼있으면 안돼있는 상태로 그냥 어플리케이션화면으로 다시 돌아간다.

## Implicit and Hybrid Flow

default로, JavaScript adapter는 Authorization Code 플로우를 사용한다.

이 플로우를 통해 Keycloak 서버는 authentication code가 아닌 authorization code를 어플리케이션에 반환한다. JavaScript adapter는 브라우저가 어플리케이션으로 redirected 된 후에 code를 access token이나 refresh token으로 교환한다.

Keycloak은 인증과정 성공 후 access token이 바로 보내지는 implicit flow도 지원한다. 이건 토큰을 위한 추가적인 코드 교환이 이루어지지 않기 때문에 기존 플로우보다 더 나은 성능을 보여줄 수 있지만, access token이 만료됐을 때 영향을 줄 수 있다.

access token을 url에 포함시켜 보내는 것은 보안상 취약할 수 있다. 예를 들어 웹 서버 로그나 브라우저 히스토리를 통해 토큰이 노출 될 수 있다.

implicit 플로우를 활성화 시키려면, Keycloak Administration Console에서 Implicit Flow Enabled 플래그를 활성화시키면 된다. 아니면 init 시점에서 flow파라미터를 implicit으로 설정하면 된다.

```jsx
keycloak.init({
    flow: 'implicit'
})
```

하나 주의해야 할 점은 access token만 제공되고 refresh token은 제공되지 않는다는 점이다. 이것은 access token이 만료되면 다시 Keycloak으로 redirect 해서 새로운 access token을 받아야 된다는 것을 의미한다.

Keycloak은 Hybrid flow도 지원한다.

사용자는 Standard Flow Enabled와 Implicit Flow Enabled 플래그 모두를 admin console에서 활성화 시켜야 된다. 그러면 Keycloak 서버는 code와 token 모두를 어플리케이션에 보낸다. access token은 바로 사용될 수 있고, code는 access token이나 refresh token으로 교환될 수 있다. implicit 플로우와 비슷하게, hybird 플로우는 acceess token을 바로 받아 쓸 수 있기 때문에 성능면에서 좋다. 하지만 토큰이 url 안에 포함돼서 보내지기 때문에 보안상 취약할 수 있다.

Hybird 플로우의 한가지 장점은 refresh token을 만들 수 있다는 것이다. init시점에서 flow파라미터를 hybrid로 설정하면 된다.

## Modern Browsers with Tracking Protection

몇몇 브라우저의 최신 버전에선 크롬의 SamSite 같은 third파티로부터 사용자가 추적당하는 것을 방지하거나, third파티 쿠키를 완전히 차단시키기 위해 다양한 쿠키 정책들이 적용되었다. 이러한 정책들은 앞으로 더 엄격해지고 더 많은 브라우저에 적용될 것으로 예상된다. third파티 context의 쿠키들이 완전히 지원이 안되게끔 하거나 브라우저에 의해 차단되는 결과를 야기할 수 있다. 미래엔 adapter 기능들이 이것의 영향을 받거나 deprecated 될 수도 있다.

### Browsers with Blocked Third-Party Cookies

Js adapter에 의해 브라우저의 행동이 감지되면 Session Status iframe은 지원되지 않고 자동으로 disabled된다. 이것은 adapter가 Single Sign-Out 감지를 위해 session cookie를 사용하지 못하고, 전적으로 token에 의존해야 된다는 것을 의미한다. 사용자가 다른 창에서 로그아웃을 하면, JavaScript adapter를 사용하는 어플리케이션은 access token을 refresh하기 전까지는 로그아웃 되지 않는다. 그렇기 때문에, Access token의 수명을 비교적 짧게 설정해서 로그아웃을 좀 더 빨리 감지하게끔 하는 것을 권장한다. (참고: [https://www.keycloak.org/docs/latest/server_admin/#_timeouts](https://www.keycloak.org/docs/latest/server_admin/#_timeouts))

slient check-sso는 지원되지 않으면 default로는 regular(non-silent) check-sso로 falls back된다. 이 처리는 init메소드에서 넘어가는 silentCheckSsoFallback 파라미터를 false로 해줌으로써 끌 수 있다. 이 경우엔, 만약 엄격한 브라우저의 행동이 감지되면 check-sso는 완전히 disabled된다.

regular check-sso 또한 영향을 받는다. Session Status iframe이 지원되지 않기 때문에, adpater가 사용자의 로그인 상태에 대해 체크하도록 initialized 됐다면 추가적인 Keycloak으로의 redirection이 이루어져야 한다. 이것은 사용자가 로그인 됐는지 확인하고 로그아웃 됐을 때만 redirect해주는, iframe이 사용되는 standard한 작업과는 차이가 있다. (13.1.으로 시작하는 Safari 등에서 영향을 받는다)

## JavaScript Adapter Reference

### **Constructor**

`new Keycloak();`  
`new Keycloak('http://localhost/keycloak.json');`  
`new Keycloak({ url: 'http://localhost/auth', realm: 'myrealm', clientId: 'myApp' });`  

### **Properties**

**authenticated** Is `true` if the user is authenticated, `false` otherwise.

**token** The base64 encoded token that can be sent in the `Authorization` header in requests to services.

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

method와 options에 대한 reference는 원본 사이트 참고

[Securing Applications and Services Guide](https://www.keycloak.org/docs/latest/securing_apps/#javascript-adapter-reference)