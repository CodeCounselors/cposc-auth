## Authorization!
### Also Known As

- Identity and Access Management (IAM)
- Role Based Access Control
- Access Control List (ACL)
- Claims Based

SLIDE
## Goals
    
1. Maintainability (No boilerplate)
1. Good separation of concerns
1. Scalability
1. Active Community
1. Open Source!
    
SLIDE
## Options

- Frameworks (Sails, Loopback, etc)
- [Node-ACL](https://github.com/OptimalBits/node_acl) <!-- .element: class="fragment highlight-green" -->
- [Must Be](https://github.com/derickbailey/mustbe)
- [CanCan](https://github.com/vadimdemedes/cancan)
    
SLIDE
## Node-ACL :  Roles
### acl.js

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

SLIDE
## Node-ACL :  Users
```javascript
return user.load().then(u => {
      const userId = u.id
      const personId = u.personId
      const individualUserRole = userId
      return user.loadOrgs().then(orgs => {
        // Setup User Roles
        const userOrgRoles = getUserOrgRoles(orgs)
        acl.addUserRoles(authId, [individualUserRole, ...userOrgRoles])
        // Authorize each Role
        const userOwnedRoutes = [`users/auths/${authId}`, `users/${authId}`, `users/${userId}`, 
                                 `people/${personId}`]
        const fullRoutes = _.map(userOwnedRoutes, r => '/api/' + r)
        acl.allow(individualUserRole, fullRoutes, '*')
      })
  })
```