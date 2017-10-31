## Authorization!
### Also Known As

- Identity and Access Management (IAM)
- Role Based Access Control
- Access Control List (ACL)
- Claims Based

VSLIDE
## Goals
    
1. Maintainability (No boilerplate) <!-- .element: class="fragment" -->
1. Good separation of concerns <!-- .element: class="fragment" -->
1. Scalability <!-- .element: class="fragment" -->
1. Active Community <!-- .element: class="fragment" -->
1. Open Source! <!-- .element: class="fragment" -->
    
VSLIDE
## Options

- Frameworks (Sails, Loopback, etc)
- [Node-ACL](https://github.com/OptimalBits/node_acl) <!-- .element: class="fragment highlight-green" -->
- [Must Be](https://github.com/derickbailey/mustbe)
- [CanCan](https://github.com/vadimdemedes/cancan)
    
VSLIDE
## Lets see it in action
### https://cposc-acl.pictoriald.com

VSLIDE
## Node-ACL :  Roles
### security/acl.js
```javascript
const ACL = require('acl')
const acl = new ACL(new ACL.memoryBackend(), logger)

const orgAdminRole = orgId => `org-admin-role-${orgId}`
const orgRole = orgId => `org-role-${orgId}`
const allowOrgRole = (orgId) => {
	acl.allow(orgRole(orgId),      `/api/organizations/${orgId}`,          'get')
	acl.allow(orgAdminRole(orgId), `/api/organizations/${orgId}/families`, '*')
}

queries.getAllOrganizations().then(orgs => orgs.forEach(o => {
  allowOrgRole(o.id)
}))
```

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
// Over in app.js: app.use('/api', apiMiddleware, api) 
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

### Remember how we defined the roles <!-- .element: class="fragment" --> 
```javascript 
const allowOrgRole = (orgId) => {
	acl.allow(orgRole(orgId),      `/api/organizations/${orgId}`,          'get')
	acl.allow(orgAdminRole(orgId), `/api/organizations/${orgId}/families`, '*')
}
```
<!-- .element: class="fragment" -->
