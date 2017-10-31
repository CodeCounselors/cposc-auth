SLIDE
# Implementing Auth0 Authentication

SLIDE
## The Auth0 Client Object
### AuthService.js

```javascript
import auth0 from 'auth0-js'
...
export default class AuthService extends EventEmitter {

	auth0 = new auth0.WebAuth({
		domain: Auth.Auth0Domain,
		clientID: Auth.Auth0ClientId,
		redirectUri: `${window.location.origin}/login`,
		audience: 'https://codec.auth0.com/userinfo',
		responseType: 'token id_token',
		//http://openid.net/specs/openid-connect-core-1_0.html#ScopeClaims
		scope: 'openid email profile'
	})
	...
}
```
<!-- .element class="stretch" data-trim contenteditable -->

SLIDE
## Authentication Object (client state)
###AuthService.js

```javascript
export default class AuthService extends EventEmitter {
...
	login() {
		this.auth0.authorize()
	}
	setSession(authResult) {
		localStorage.setItem('access_token', authResult.accessToken)
		localStorage.setItem('id_token', authResult.idToken)
	}
	logout() {
		localStorage.removeItem('access_token')
		localStorage.removeItem('id_token')
	}
	getProfile() {
		return localStorage.getItem('profile')
	}
}
```

SLIDE
## Authentication Triggers<
### routes.js
```jsx
...
	const requireAuth = (nextState, replace) => {
		if (!auth.loggedIn()) { replace({pathname: '/login'}) }
	}
	const handleAuthentication = (nextState, replace) => {
		if (/access_token|id_token|error/.test(nextState.location.hash)) {
			auth.handleAuthentication()
		}
	}
	export const makeRoutes = () => {
		return (
			&lt;Route path="/" component={Container} auth={auth}&gt;
				&lt;IndexRedirect to="/home"/&gt;
				&lt;Route path="login" component={Login} onEnter={handleAuthentication} /&gt;
				&lt;Route path="home" component={Home} onEnter={requireAuth}/&gt;
			&lt;/Route&gt;
		)
	}
}
```
	
SLIDE			
## TODO: Add Sequence Diagram<
- Request for /home
- No token so trigger login
- Auth0 calls ack to /login with tokens
- Fetch profile
- Store/Create User
					
SLIDE
## Sending the JWT
### HttpClient.js

```javascript
import axios from 'axios'

axios.interceptors.request.use(function (config) {
	const token = localStorage.getItem('id_token')
	config.headers['Authorization'] = 'Bearer ' + token
	return config
})

export default axios
```

SLIDE
## Receiving the JWT

```javascript
const jwt = require('express-jwt')
const jwksRsa = require('jwks-rsa')
const app = express()
...
const apiMiddleware = [
	jwt( {
		secret: jwksRsa.expressJwtSecret({
			cache: true,
			rateLimit: true,
			jwksRequestsPerMinute: 5,
			jwksUri: `https://${auth0Domain}/.well-known/jwks.json`
		})
	}).unless({path: [imgRoute]}),
	...
]
app.use('/api', apiMiddleware, api)
```
<!-- .element class="stretch" data-trim contenteditable -->

SLIDE
## Lets see it in action</h2>
### https://cposc-jwt.pictoriald.com
## What's Missing? <!-- .element: class="fragment" -->