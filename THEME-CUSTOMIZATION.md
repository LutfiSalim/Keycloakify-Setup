# Keycloak Theme Customization Guide

This guide provides step-by-step instructions for customizing the UI of each Keycloak theme type: login, account, admin, and email.

## Table of Contents
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Login Theme Customization](#login-theme-customization)
- [Account Theme Customization](#account-theme-customization)
- [Admin Theme Customization](#admin-theme-customization)
- [Email Theme Customization](#email-theme-customization)
- [Building and Deploying](#building-and-deploying)
- [Testing Your Theme](#testing-your-theme)

## Project Structure

This project uses [Keycloakify](https://github.com/keycloakify/keycloakify), a framework that allows you to build Keycloak themes using React. The project structure is as follows:

```
src/
├── account/       # Account theme customization
├── admin/         # Admin theme customization
├── email/         # Email theme customization
├── login/         # Login theme customization
├── shared/        # Shared components and styles
├── kc.gen.tsx     # Generated Keycloak context
└── main.tsx       # Entry point
```

## Getting Started

Before making any changes, make sure you have the following prerequisites:

1. Node.js (version 18.x or 20.x)
2. npm or yarn
3. A running Keycloak instance for testing

To start development:

```bash
# Install dependencies
npm install

# Start the development server
npm run dev
```

## Login Theme Customization

The login theme controls the appearance of all authentication-related pages (login, registration, forgot password, etc.).

### Step 1: Understand the structure

The login theme is located in `src/login/` and typically contains:
- `KcContext.ts` - Type definitions for the login context
- `KcPage.tsx` - The main component that wraps all login pages
- `pages/` - Individual page components

### Step 2: Customize the login pages

1. **Modify existing pages**:
   Navigate to `src/login/pages/` and edit the React components for specific pages.

   Example for customizing the login page:
   ```tsx
   // src/login/pages/Login.tsx
   import { useKcContext } from "../KcContext";
   
   export default function Login() {
     const { kcContext, i18n, doUseDefaultCss, Template } = useKcContext();
     
     return (
       <Template
         {...{ kcContext, i18n, doUseDefaultCss }}
         headerNode={<h1>My Custom Login</h1>}
       >
         {/* Your custom login form */}
       </Template>
     );
   }
   ```

2. **Add custom CSS**:
   Create a CSS file in the login directory and import it in your components.

   ```tsx
   import "./login.css";
   ```

3. **Add custom assets**:
   Place images, fonts, and other assets in the `public` directory and reference them in your components.

### Step 3: Register your custom pages

Make sure your custom pages are properly registered in the `KcPage.tsx` file:

```tsx
// src/login/KcPage.tsx
import { KcPageProps } from "keycloakify";
import Login from "./pages/Login";

export default function KcPage(props: KcPageProps) {
  const { kcContext, ...kcProps } = props;

  switch (kcContext.pageId) {
    case "login.ftl": return <Login {...{ kcContext, ...kcProps }} />;
    // Add more custom pages here
    default: return <Default {...{ kcContext, ...kcProps }} />;
  }
}
```

## Account Theme Customization

The account theme controls the user account management console.

### Step 1: Understand the structure

The account theme is located in `src/account/` and uses the `@keycloakify/keycloak-account-ui` package.

### Step 2: Customize the account pages

1. **Create a custom theme**:
   ```tsx
   // src/account/index.tsx
   import { createAccountTheme } from "@keycloakify/keycloak-account-ui";
   
   export const { createPageStory } = createAccountTheme({
     // Theme customization options
     extraThemeProperties: {
       // Custom CSS variables
       "--my-custom-color": "#ff0000",
     },
     // Custom components
     components: {
       // Override specific components
       Header: ({ defaultHeader }) => (
         <div>
           <h1>My Custom Account Header</h1>
           {defaultHeader}
         </div>
       ),
     },
   });
   ```

2. **Customize specific pages**:
   Override specific account pages by providing custom components in the `components` object.

3. **Add custom styles**:
   Use the `extraThemeProperties` option to define custom CSS variables.

## Admin Theme Customization

The admin theme controls the appearance of the Keycloak administration console.

### Step 1: Understand the structure

The admin theme is located in `src/admin/` and uses the `@keycloakify/keycloak-admin-ui` package.

### Step 2: Customize the admin console

1. **Create a custom theme**:
   ```tsx
   // src/admin/index.tsx
   import { createAdminTheme } from "@keycloakify/keycloak-admin-ui";
   
   export const { createPageStory } = createAdminTheme({
     // Theme customization options
     extraThemeProperties: {
       // Custom CSS variables
       "--my-admin-color": "#0066cc",
     },
     // Custom components
     components: {
       // Override specific components
       Header: ({ defaultHeader }) => (
         <div>
           <h1>My Custom Admin Header</h1>
           {defaultHeader}
         </div>
       ),
     },
   });
   ```

2. **Customize specific sections**:
   Override specific admin components by providing custom components in the `components` object.

3. **Add custom styles**:
   Use the `extraThemeProperties` option to define custom CSS variables.

## Email Theme Customization

The email theme controls the appearance of emails sent by Keycloak.

### Step 1: Understand the structure

The email theme is located in `src/email/` and uses the `@keycloakify/email-native` package.

### Step 2: Customize email templates

1. **Create custom email templates**:
   ```tsx
   // src/email/index.tsx
   import { createEmailTheme } from "@keycloakify/email-native";
   
   export const { createPageStory } = createEmailTheme({
     // Email customization options
     extraThemeProperties: {
       // Custom email properties
       brandName: "My Company",
       brandUrl: "https://example.com",
     },
     // Custom email templates
     components: {
       // Override specific email templates
       emailVerification: ({ defaultEmailVerification, kcContext }) => (
         <div>
           <h1>Please Verify Your Email</h1>
           <p>Click the link below to verify your email address:</p>
           {defaultEmailVerification}
         </div>
       ),
     },
   });
   ```

2. **Customize specific email templates**:
   Override specific email templates by providing custom components in the `components` object.

3. **Add custom styles**:
   Email templates should use inline CSS for maximum compatibility with email clients.

## Building and Deploying

To build your custom themes for deployment to Keycloak:

```bash
# Build the Keycloak theme
npm run build-keycloak-theme
```

This will generate a JAR file in the `dist_keycloak` directory that you can deploy to your Keycloak server.

### Deploying to Keycloak

1. Copy the generated JAR file to the `providers` directory of your Keycloak installation:
   ```bash
   cp dist_keycloak/keycloakify-starter-*.jar /path/to/keycloak/providers/
   ```

2. Restart your Keycloak server to apply the changes.

## Testing Your Theme

### Testing in Development Mode

During development, you can preview your themes using Storybook:

```bash
npm run storybook
```

This will start a Storybook server where you can preview all your theme components.

### Testing with a Real Keycloak Instance

To test your theme with a real Keycloak instance:

1. Build and deploy your theme as described above.
2. Log in to the Keycloak admin console.
3. Navigate to the realm you want to customize.
4. Go to Realm Settings > Themes.
5. Select your custom theme for each theme type (login, account, admin, email).
6. Save your changes and test the theme.

## Troubleshooting

- **Theme not showing up**: Make sure the JAR file is properly deployed and Keycloak has been restarted.
- **CSS not applying**: Check that your CSS selectors are specific enough and not being overridden.
- **JavaScript errors**: Check the browser console for errors and make sure your components are properly exported.

## Additional Resources

- [Keycloakify Documentation](https://github.com/keycloakify/keycloakify)
- [Keycloak Themes Documentation](https://www.keycloak.org/docs/latest/server_development/#_themes)
- [PatternFly Documentation](https://www.patternfly.org/v4/) (Keycloak's default UI framework)
