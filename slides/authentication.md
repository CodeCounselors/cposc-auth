<!-- .element data-background-image="img/apple-desk-laptop-macbook.jpg" -->
## https://cposc.pictoriald.com
# What's Missing?
						
VSLIDE
<!-- .element data-background-image="img/apple-desk-laptop-macbook.jpg" -->
## Authentication!
### Answers the question: <!-- .element style="color:#859900" class="fragment" -->
Are you who you say you are? <!-- .element: class="fragment" -->

VSLIDE
<!-- .element data-background-image="img/photo-1424764016210-9482f49dde7f.jpeg" -->
## What do we need?

1. Login Form <!-- .element: class="fragment" -->
1. User Datastore <!-- .element: class="fragment" -->
1. Password Management <!-- .element: class="fragment" -->
    - Email Verification <!-- .element: class="fragment" -->
    - Password Reset <!-- .element: class="fragment" -->
    - Account Lockout <!-- .element: class="fragment" -->
    - Multi-Factor Authentication (MFA) <!-- .element: class="fragment" -->
    - Passwordless login <!-- .element: class="fragment" -->

note:
- Before fragments: Remember the 3 main goals?

VSLIDE
<!-- .element data-background-image="img/photo-1424764016210-9482f49dde7f.jpeg" -->
## What options do we have?
1. Roll your own <!-- .element: class="fragment" -->
2. Don't roll your own (all on your own) <!-- .element: class="fragment" -->
    - Devise (Ruby)
    - Passport (NodeJS)
    - Spring Security (JVM)
3. Don't roll your own at all <!-- .element: class="fragment" -->
    - Auth0
    - Okta (Stormpath)
						
VSLIDE
<!-- .element data-background-image="img/pexels-photo-325229-servers.jpeg" style="color:white" -->
#   
## JWT = JSON Web Token <!-- .element: class="fragment"  style="color:white" -->
1. Header <!-- .element: class="fragment" -->
1. Payload <!-- .element: class="fragment" -->
1. Signature <!-- .element: class="fragment" -->


VSLIDE
<!-- .element data-background-image="img/pexels-photo-325229-servers.jpeg" style="color:white" -->
![JWT.IO](img/jwtio.png)
<!-- .element class="stretch" data-trim contenteditable -->

note:
- How JWT was captured
- How is a JWT Generated...

VSLIDE
<!-- .element data-background-image="img/pexels-photo-325229-servers.jpeg" style="color:white" -->
### JWT in Action <!-- .element: style="color:#FFF" -->
![Auth0 Handshake](img/auth0-seq.png)

VSLIDE
<!-- .element data-background-image="img/pexels-photo-325229-servers.jpeg" -->
## Simple as that, right? <!-- .element: style="color:white" -->
- Are JWTs better than stateful sessions? <!-- .element: class="fragment" style="color:white" -->
- JWT Signing Options <!-- .element: class="fragment" style="color:white" -->
    - HS256; Symetric (Shared Secrets) <!-- .element: class="fragment" style="color:white" -->
    - RS256; Assymetric (Public/Private Keys) <!-- .element: class="fragment" style="color:white"-->
   
   
    
SLIDE
# Implementing Auth0 Authentication

VSLIDE
## The Auth0 Client Object
### AuthService.js

```javascript
import auth0 from 'auth0-js'

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

VSLIDE
## Authentication Object (client state)
### AuthService.js

```javascript
export default class AuthService extends EventEmitter {
...
	login() {
		this.auth0.authorize()
	}
	setSession(authResult) {
		localStorage.setItem('id_token', authResult.idToken)
	}
	logout() {
		localStorage.removeItem('id_token')
	}
	getProfile() {
		return localStorage.getItem('profile')
	}
}
```
<!-- .element class="stretch" data-trim contenteditable -->

VSLIDE
## Authentication Triggers
### routes.js
```jsx
import {Route, IndexRedirect} from 'react-router'

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
        <Route path="/" component={Container} auth={auth}>
            <IndexRedirect to="/home"/>
            <Route path="login" component={Login} onEnter={handleAuthentication} />
            <Route path="home" component={Home} onEnter={requireAuth}>
        </Route>
    )
}
```
<!-- .element class="stretch" data-trim contenteditable -->
	
VSLIDE
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

VSLIDE
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

VSLIDE
## Lets see it in action
### https://cposc-jwt.pictoriald.com
#### username: demo@pictoriald.com
#### password: demo
## What's Missing? <!-- .element: class="fragment" -->		

SLIDE
<!-- .element data-background-image="img/photo-1424764016210-9482f49dde7f.jpeg" -->
## Authorization!
### Answers the question: <!-- .element style="color:#859900" class="fragment" -->
Are you permitted to perform the action you are attempting? <!-- .element: class="fragment" -->

### Also Known As<!-- .element: class="fragment" -->

- Identity and Access Management (IAM) <!-- .element: class="fragment" -->
- Role Based Access Control <!-- .element: class="fragment" -->
- Access Control List (ACL) <!-- .element: class="fragment" -->
- Claims Based <!-- .element: class="fragment" -->


VSLIDE
<!-- .element data-background-image="img/photo-1424764016210-9482f49dde7f.jpeg" -->
## Goals
    
1. Maintainability (No boilerplate) <!-- .element: class="fragment" -->
1. Good separation of concerns <!-- .element: class="fragment" -->
1. Scalability <!-- .element: class="fragment" -->
1. Active Community <!-- .element: class="fragment" -->
1. Open Source! <!-- .element: class="fragment" -->
    
VSLIDE
<!-- .element data-background-image="img/photo-1424764016210-9482f49dde7f.jpeg" -->
## Options

- Frameworks (Sails, Loopback, etc)
- [Node-ACL](https://github.com/OptimalBits/node_acl) <!-- .element: class="fragment highlight-green" -->
- [Must Be](https://github.com/derickbailey/mustbe)
- [CanCan](https://github.com/vadimdemedes/cancan)
    
VSLIDE
## Lets see it in action
### https://cposc-acl.pictoriald.com
#### username: demo@pictoriald.com
#### password: demo

VSLIDE
## Node-ACL :  Roles
### security/acl.js
```javascript
const ACL = require('acl')
const acl = new ACL(new ACL.memoryBackend(), logger)

// Construct Organization Role Names
const orgAdminRole = orgId => `org-admin-role-${orgId}`
const orgRole = orgId => `org-role-${orgId}`

// Map each role to a API Route
const allowOrgRole = (orgId) => {
	acl.allow(orgRole(orgId),      `/api/organizations/${orgId}`,          'get')
	acl.allow(orgAdminRole(orgId), `/api/organizations/${orgId}/families`, '*')
}

// Scan the database for organizations and setup the ACLs for each
queries.getAllOrganizations().then(orgs => orgs.forEach(o => {
  allowOrgRole(o.id)
}))
```
<!-- .element class="stretch" data-trim contenteditable -->

VSLIDE
## Node-ACL :  Users
### security/acl.js
```javascript
return user.load().then(user => {
      const {userId, personId} = user
      const individualUserRole = userId
      return user.loadOrgs().then(orgs => {
        // Setup User Roles
        const userOrgRoles = getUserOrgRoles(orgs)
        acl.addUserRoles(authId, [individualUserRole, ...userOrgRoles])
        // Authorize each Role
        const userOwnedRoutes = [`users/auths/${authId}`, `users/${userId}`, `people/${personId}`]
        const fullRoutes = _.map(userOwnedRoutes, route => '/api/' + route)
        acl.allow(individualUserRole, fullRoutes, '*')
      })
  })
```

VSLIDE
## Node-ACL : Route Middleware
### routes/api.js
```javascript
import acl from '../security/acl'

// No Middleware - Why?
router.get('/users/auths/:authId', (req, res, next) => {...})

// ACL Middleware, entire route 
router.put('/users/:userId', acl.middleware(), (req, res, next) => {...}) 

// ACL Middleware, route wildcarded
router.get('/organizations/:orgId/families', acl.middleware(3), (req, res, next) => {...})
```
    

VSLIDE
## Route Design for Role Based Security

### Everything from the previous slide, plus...

```javascript
import acl from '../security/acl'

// A set of routes with security based on the orgId
router.post('/organizations/:orgId/families/:familyId/familymembers',  acl.middleware(4), ...
router.delete('/organizations/:orgId/families/:familyId/familymembers/:personId',  acl.middleware(4), ...
router.delete('/organizations/:orgId/families/:familyId',  acl.middleware(4), ... 

```

### Remember how we defined the roles: <!-- .element: class="fragment" --> 
```javascript 
const allowOrgRole = (orgId) => {
	acl.allow(orgRole(orgId),      `/api/organizations/${orgId}`,          'get')
	acl.allow(orgAdminRole(orgId), `/api/organizations/${orgId}/families`, '*')
}
```
<!-- .element: class="fragment" -->

VSLIDE
# Where you expecting more?

SLIDE
# Summary
### Don't Roll your own
1. Authentication - Confirm User Identity
1. Authorization - Confirm User Access
    - Group Privileges into Roles
    - Assign Roles to Users
    - Assign Roles to Operations


VSLIDE
## Resources
- https://auth0.com/blog/cookies-vs-tokens-definitive-guide/
- https://auth0.com/blog/navigating-rs256-and-jwks/

VSLIDE
## Thank You!
### @codecounselor
### https://codecounselors.github.io/cposc-auth
