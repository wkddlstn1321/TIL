# OAuth 2.0

Oauth 프로토콜은 인증 및 인가를 위한 프로토콜로 제 3자 앱에서 사용자의 데이터에 접근할 수 있는 권한을 부여해준다.

제 3자 앱 이 구글이라고 할 때 유저는 구글 로그인을 통해 구글 유저임을 인증하고 권한 허용을 통해 구글에서 api로 제공하는 유저의 일부 정보를 가져다 쓸 수 있게 해준다

사용자 이름과 암호를 공유하지 않고도 액세스를 관리할 수 있기 때문에 보안적으로 훌륭

종류도 여러가지가 있지만 가장 보편전인 Authorization Code 방식으로 실제로 구현한 간편 로그인을 예시로 들며 설명하겠다.

**Authorization_Code**

전체적인 흐름
1. 유저에 요청 시 클라이언트는 권한 서버에서 제공하는 로그인페이지를 브라우저에 출력
2. 유저가 해당 페이지에서 로그인하면 권한 서버는 Authorization Code를 redirect_url로 전송
3. code 는 권한 서버에서 제공하는 API를 통해 Access Token으로 교환됨

리소스 소유자인 유저가 인증 서버에 로그인하여 인증을 진행하고 권한이 있는 코드
클라이언트는 해당 코드를 이용해 액세스 토큰을 리소스 서버로 부터 부여받고
액세스 토큰을 이용해 리소스 서버에 api 요청을 보내면 원하는 데이터를 얻을 수 있다.


실제 구현 예시

요구사항 : 42 OAuth를 통한 간편 로그인 기능
필요 데이터 : 유저의 닉네임, 이메일

**42api 등록**

Oauth를 사용할 서버에 api에 우리의 앱을 등록해야 한다.

api를 등록하면 UID와 secret, uri를 제공해주고 redirect_uri를 정할 수 있게 해준다.
UID는 우리가 등록한 앱의 식별자 이다.
sercet은 우리가 등록한 앱에서 요청을 보낼 때 사용할 비밀번호로 외부에 노출하면 안된다. 

어디에 사용하나? 천천히 알아보자


**인증**

api등록을 완료하면서 알려준 uri는 로그인 페이지로 이어진다. 

유저가 로그인 버튼을 눌렀을 때 우리는 저 페이지를 노출시켜주면 된다.

``https://api.intra.42.fr/oauth/authorize?client_id=u-s4t2ud-801649286a9bd1a88b32e7f924abb1e0439fff57438b05157c59b7a6ef8c98a1&redirect_uri=http%3A%2F%2F192.168.219.104%3A3000%2Fredirect&response_type=code`` 

이때 쿼리스트링으로 client_id와 redirect_uri를 넘겨줘야하는데 각 의미를 알아보자

* client_id
	
	api를 등록하면서 받았던 UID를 입력 이 UID를 통해서 어떤 앱(클라이언트)에 요청인지를 리소스 서버에서 식별한다. 쉽게 말해서 보낸사람이 누군지 알려준다고 생각하면 된다.

* redirect_uri

	로그인페이지 에서 유저가 로그인 인증을 마친 후 code와 함께 redirect될 uri를 의미한다.
	이때 앱을 등록할 때 작성했던 redirect_uri와 정확히 일치하지 않으면 에러가 발생한다.

아래는 직접 구현했던 Oauth 로그인 페이지로 이동시키는 react 코드에 일부이다.
```js
<Link
                to={`https://api.intra.42.fr/oauth/authorize?client_id=${process.env.REACT_APP_OAUTH_ID}&redirect_uri=${process.env.REACT_APP_OAUTH_REDIRECT_URI}&response_type=code`}
            >
                <button style={{ width: '200px', height: '50px', display: 'flex', alignItems: 'center', justifyContent: 'center'}}>
                    <img
                        src='/images/42Logo.png'
                        style={{ marginRight: '10px', width: '30px', height: '30px' }}
                    />
                    {/* 42Seoul Login */}
                     <span style={{ flex: 10, margin: '0 10px'}}>42Seoul Login</span>
                </button>
            </Link>
```
이제 유저가 로그인을 마친후 권한 부여 동의를 하면 우리가 지정한 redirect_uri로 code가 넘어올거다.

**권한 토큰 발급**

넘어온 code를 이용해 AccessToken을 발급받아야 유저의 정보를 요청해서 얻어올 수 있다. 

이제부터 중요하다.

우선 redirect_uri는 프론트에서 받을 수 있게 처리해놨다.
```js
const OAuth = () => {
    const [searchParams] = useSearchParams();
    const code = searchParams.get('code');
	...
	...
}
```

OAuth 컴포넌트로 정보가 넘어오고 code를 추출했다.

이제 두 가지 방식이 있다

1. 프론트에서 code를 이용해서 바로 accessToken을 발급받고 이걸 백으로 준다.
   
2. 프론트에서 code를 백으로 주고 백에서 accessToken을 발급받는다.

두 가지 방식있다고 했지만 사실 1번은 못쓰는 방식이다.

나도 처음에는 1번 방식으로 구현을 했었는데 이건 몇가지 문제가 있다.

* 보안
	
	직접 액세스 토큰을 발급하는 것은 클라이언트가 보안 키와 같은 중요한 정보를 저장하고 처리해야 함으로 보안상 불리하다. 중요한 정보는 백에서 처리하는게 보안적으로 안전한 방식이라고 볼 수 있겠다.

* 액세스 토큰 유효기간 관리
	
	엑세스 토큰에는 유효기간이 있다. 토큰이 만료되면 재발급을 받아야 하는데 프론트에서 유효기간 관리 로직을 추가하는건 구조상 아름답지 못하다. 백엔드에서 토큰 관리등의 로직 부분을 담당 해주는게 효율적이라고 볼 수 있겠다. 

* 정책

	사실 이게 제일 컸다. 보안도 신경안쓰고 효율도 신경 안쓴다면 프론트에서 aceessToken사용 못할건 없다. 근데 일부 OAuth 제공 업체는 보안 정책상 클라이언트에서 직접 액세스 토큰을 발급 받는걸 허용하지 않을 수 있는데 42Oauth 가 그러했다. 덕분에 문제가 발생해서 2번 방안으로 틀을 다시 잡을 수 있었다.

아래는 code를 백엔드로 넘기는 코드이다.

```js
const fetchOauth = async ({ code }) => {
    const response = await axios.post(`http://${process.env.REACT_APP_IP_ADDRESS}:4000/user/signin`,
        { code },
        { withCredentials: true },
    );
    return (response);
}
```

이제 백엔드에서 code를 이용해서 accessToken을 발급받아야 한다.
이때 secret을 사용한다.

``    const url = `https://api.intra.42.fr/oauth/token?grant_type=authorization_code&client_id=${process.env.REACT_APP_OAUTH_ID}&client_secret=${process.env.REACT_APP_OAUTH_SECRET}&code=${code}&redirect_uri=${process.env.REACT_APP_OAUTH_REDIRECT_URI}`;
``

리소스 서버에서는 uid로 클라이언트를 식별하고 secret과 code를 검증하여 올바른 요청인지를 체크한 후 accessToken을 발급해준다.


발급받은 accessToken을 header에 Authorization bearer타입으로 요청에 같이 보내면 OAuth에서 api로 제공해주는 유저의 정보를 요청할 수 있다.

백엔드 코드
```js
const headers = {
      Authorization: `Bearer ${data.access_token}`,
    };
    const _url = 'https://api.intra.42.fr/v2/me';
    const response = await firstValueFrom(
      this.httpService.get(_url, { headers }).pipe(
        catchError((error: AxiosError) => {
          
          if (error.response)
            throw new HttpException(error.response.data, error.response.status);
          else throw new InternalServerErrorException();
        }),
      ),
    );
```

전체적인 흐름에 대해서 알아봤다.

Oauth 방식이나 제공업체 조금씩 사용방법이 다를 수 있지만 큰 틀은 같을 것이다.

