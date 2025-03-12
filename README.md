# React + Azure AD Starter

This start app integrates the following features:

- Enforces RBAC at the component or route level using the fantastically
  useful [Microsoft Authentication Library for
  React](https://www.npmjs.com/package/@azure/msal-react)
- Uses the beautiful [Microsoft Graph Toolkit for
  React](https://www.npmjs.com/package/@microsoft/mgt-react) log-in widgets
- Optionally, initializes user login on page load for a smoother protected-app experience
- Demonstrates protected routes using popular [React Router](https://reactrouter.com/en/main) library
- Use code splitting with [Loadable Components](https://loadable-components.com/) for ultra-fast page loads
- Standard navbar with vanilla CSS
- Uses TypeScript React (`.tsx`)

## Why this starter?

- The awesome `@azure/msal-react` and `@microsoft/mgt-react` libraries don't
  well play together without some trickery
- Private and protected routes and components should just work (without lots
  of boiler plate code!)
- Using user access tokens should be easy (without lots of boiler plate code!)

## Getting started:

1. Create an Azure AD app registration, and collect the Client ID, Tenant ID,
   and Scope in a `.env` file at the root of his repo, like so:

   ```txt
   REACT_APP_CLIENT_ID={{Azure AD app client ID}}
   REACT_APP_AUTHORITY=https://login.microsoftonline.com/{{Azure AD tenant ID}}
   REACT_APP_SCOPE={{ Scope URI }}
   ```

1. Install the dependencies with `yarn` and start the app with `yarn start`,
   and view it at [localhost:3000](), as usual.

1. View the examples:

- Forced user login `src/App.tsx`
- Private and protected routing in `src/main.tsx`
- Private and protected components in `src/components/page-1/index.tsx`
- Using user access tokens in `src/components/page-1/index.tsx`

## Using User Access Tokens

User Access Tokens can be obtained in the app using the
`fetchUserAccessToken` function , typically for fetching protected content
from a protected API

```js
const [protectedAPIContent, setProtectedAPIContent] = useState();

useEffect(async () => {
  const userAccessToken = await fetchUserAccessToken();
  const response = await fetch("/some/protected/route.json", {
    headers: {
      Authorization: `Bearer ${userAccessToken}`,
    },
  });

  // do things with the protected contents
  setProtectedAPIContent(await response.json());
});
```

## Restricting Content to using User Authentication

### Manual control

To conditionally render components manually, the `useUserAccount()` hook provides access to the user account and the the users assigned roles.

```jsx
import { useUserAccount } from "src/auth/UserAccount";

const MyAwesomeComponent = () => {
  const AccountInfo = useUserAccount();
  return (
    <span>
      You are{" "}
      {"admin" in (AccountInfo.roles || []) ? "Authorized" : "Unauthorized"}
    </span>
  );
};
```

However, the following components will typically offer a more thorough
Auth/Auth controlled rendering.

### Private Content

The `PrivateComponent` requires that a user is signed in in order to view the
private contents.

```jsx
<PrivateComponent
  unauthenticated={() => <p>Loading...</p>}
  loading={() => <p>Loading...</p>}
  content={() => <div>You must be logged in to view this component</div>}
/>
```

### Protected Content

The `RBACProtectedComponent` requires that a user is signed in _and_ is
assigned a role, as follows

- When the `requiredRole` attribute is a string, the user is required to be
  assigned a role in order to view the protected content:

  ```jsx
  <RBACProtectedComponent
    requiredRole="RequiredRole" // string
    loading={() => <p>Loading...</p>}
    unauthorized={() => <p>UNAUTHORIZED</p>}
    unauthenticated={() => <p>LOGGED OUT</p>}
    content={() => (
      <p>You must be assigned the "RequiredRole" to view this component</p>
    )}
  />
  ```

- When the `requiredRole` attribute is an array of strings, the user is
  required to be assigned have _all of_ provided roles in order to view the
  protected content:

  ```jsx
  <RBACProtectedComponent
    requiredRoles={["RoleOne", "RoleTwo"]} // string[]
    loading={() => <p>Loading...</p>}
    unauthorized={() => <p>UNAUTHORIZED</p>}
    unauthenticated={() => <p>LOGGED OUT</p>}
    content={() => (
      <p>
        You must be assigned <em>both</em> "RoleTwo" and "RoleTwo" to view this
        component
      </p>
    )}
  />
  ```

- When the `allowedRoles` is provided, the user is required to have _at least
  one_ of the provided roles in order to view the protected content:

  ```jsx
  <RBACProtectedComponent
    allowedRoles={["RoleOne", "RoleTwo"]} // string[]
    loading={() => <p>Loading...</p>}
    unauthorized={() => <p>UNAUTHORIZED</p>}
    unauthenticated={() => <p>LOGGED OUT</p>}
    content={() => (
      <p>
        You must be assigned <em>either</em> "RoleTwo" and "RoleTwo" to view
        this component
      </p>
    )}
  />
  ```

Note that a simple way to enforce protected routing is to use `() =>
<Navigate to="/unauthorized" replace />` in either the `unauthorized` or
`unauthenticated` properties of an `RBACProtectedComponent`
