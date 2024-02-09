# Plura Production Setup

This document provides a step-by-step guide to setting up the Plura Production project.

## Part 1

### Necessary Installation

- Bun runtime
- MySQL Workbench
- NVM

### Project Setup

1. Start by creating the APP in the terminal:
    ```bash
    bunx create-next-app@latest .
    ```
    For the questions asked, just say “NO” to ‘import alias’. All the others questions choose “YES”.

2. Create a `.env` file ( in `app/src` ) and paste the code in the git repository inside it.

3. Configure the “uploadthing”. For this you will need to create an account at uploadthing.com and create the app. After creating the app you need to copy the API key inside your `.env` file. We are going to use “uploadthing” to upload images and files.

4. In this project we are going to use shadcn.ui: https://ui.shadcn.com/. Go to installation and choose the “bun” option that is the one we are going to use, and install it in your terminal:
    ```bash
    bunx —bun shadcn-ui@latest add
    ```
    Just press “A” in your keyboard and it will install all the components.

5. Go to your MySQL Workbench and create a database. *port 3306 if not default. Now that we have it created we need to put it into our `.env` file as the following:
    ```jsx
    DATABASE_URL="mysql://root:test@localhost:3306/PLURA-APP" // As we are using Localhost, just past it here ATM
    PROD_DATABASE_URL=
    LOCAL_DATABASE_URL="mysql://root:test@localhost:3306/PLURA-APP"
    BEXT_PUBLIC_BUILDER_API_KEY=
    // Where test= password
    // Where app_name= plura (Same name as your DB)
    ```

6. Now we are going to our `next.config.js` which is gonna look like:

    <image of config.js>

7. Lets make sure everything is running ok:
    ```bash
    bun run dev
    ```
    After that we may have a problem with upgrade nodeJS ( https://github.com/nvm-sh/nvm?tab=readme-ov-file#installing-and-updating )

    Now run: 
    ```bash
    nvm install node
    nvm use node
    ```
    NVM will be used in order to switch between Node version. ( In case you need to use an older version )… Restart VSCode.

    In a new terminal ( inside VScode ) run the app again: 
    ```bash
    bun run dev
    ```
    In NextJS works by a request, so when running “run dev” it is necessary to go into the localhostURL and just refresh the domain (F5). This will generate a request and the project will run.

8. Next step is configure the Clerk authentication. Go ahead in clerk.com and create an account

    In your clerk account go to “Add application”, deactivate the “Email adress” option and put your app name on the top. With this it will create our environmnet variable with must be past inside `.env`.
    ```bash
    NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY= <past here>
    CLERK_SECRET_KEY= <past here>
    ```

9. Now we are going to install clerk themes that will be used to darkmode and this kind of stuff. 
    ```bash
    bun add @clerk/themes 
    ```
    Inside you `.env` file add the following:
    ```bash
    NEXT_PUBLIC_URL=http://localhost:3000/
    NEXT_PUBLIC_DOMAIN=localhost:3000
    NEXT_PUBLIC_SCHEME=http://
    ```

10. Next step is going to be “uploadthing”. Visit the website and go to documentaion. In the installation packages choose BUN:
    ```bash
    bun add uploadthing @uploadthing/react
    ```

11. Now we need to change the `tailwind.config.js`. We need a typescript config file, so go to the JS file and copy everything inside and paste it inside “tailwind.config.ts”. Now we can delete “tailwind.config.js”. Now we need to import uploadthing and call the function as:
    ```tsx
    import {withUt} from 'uploadthing/tw'

    module.exports = withUt(<PASTE ALL CONTENT IN CONFIG.JS AS PARAMETER AND WRAP IT INSIDE THIS FUNCTION>)
    ```

# Part 2

## .env and .gitignore

Add the `.env` inside a `.gitignore` file to hide our `.env` file and not to push it to the repository.

## Clerk Setup

1. Go to the Clerk website and navigate to Home > Continue to docs to get the terminal installation command:
    ```bash
    bun add @clerk/nextjs
    ```

2. Choose your theme in Shadcn UI theme generator. Paste the code in `src/app/global.css` and replace everything inside. You can also add a CSS reset.

3. Update your `.env` file with the following entries:
    ```bash
    NEXT_PUBLIC_CLERK_SIGN_IN_URL=/agency/sign-in
    NEXT_PUBLIC_CLERK_SIGN_UP_URL=/agency/sign-up
    NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/
    NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/
    ```

4. Wrap this application in the Clerk provider. Update `src/app/layout.tsx` accordingly.

5. Import the dark theme:
    ```tsx
    import { ClerkProvider } from '@clerk/nextjs'
    import { dark } from '@clerk/themes'
    ```

## Routing Structure

**IMPORTANT!!**

1. Create a folder named `(main)` inside `src/app/`. This will be our route group. This is where our application will remain.

2. Inside `(main)`, create a folder named `agency` and another named `subaccount`.

3. Inside `agency`, create a folder named `(auth)`, which will contain folders `sign-in` and `sign-up`.

4. Create a dynamic route in our `sign-in/up` folder as `[[…sign-in]]`. Create a file `page.tsx` and return the `SignIn` component inside it.

5. Call the component for both `sign-in` and `sign-up`:
    ```tsx
    import { SignUp } from '@clerk/nextjs'
    import React from 'react'

    const Page = () => {
        return <SignIn />
    }

    export default Page

    // Do the same with SignUp
    ```

6. Set up the middleware file: Inside `src`, create a file `middleware.ts`. Get the `middleware.ts` code from the Clerk documentation and paste inside it.

7. Create a layout page inside the `(auth)` folder. Inside it, we are going to render a wrapper that is going to give us some new styling.

8. If you have a website landing page, create a `site` folder inside `src/app` and move the `page.tsx` inside it. Now you are able to code your landing page.

**TIPS**: 

- In the website bg.belick.com, there are snippets to help you design your landing page.
- The navbar tag needs to be inside your `layout.tsx` before `{ children }` tag.

## Subdomains Architecture

1. Let’s create a domain route: To do this, create the following files in `..app/[domain]/[path]`. After that, create a `page.tsx` for each of them (one inside `[domain]` and one inside `[path]`).

2. A bit of an explanation of Clerk provider is that inside your main layout (`src/app/layout.tsx`), all the components are wrapped inside the Clerk tag `<ClerkProvider> Your landing page code here </ClerkProvider>`, which means your full application is wrapped inside it. For this specific project, there are only 2 paths set as public URLs in `authMiddleware`, but we also want the `[domain]` and `[path]` to be public. 

3. To fix this, remove the `ClerkProvider` tag inside our root app and create a `page.tsx` inside `(main)` and return the layout and add the Clerk tag as follows:

4. Now we also need to add the Clerk provider in our `src/app/site layout.tsx` as follows:

# Part 3

## Middleware Setup

1. Go to the `middleware.ts` to add the functions before/after Auth. This is the first step of the logic behind our app navigation structure.

    ```tsx
    import { authMiddleware } from '@clerk/nextjs'
    import { NextResponse } from 'next/server'

    // This example protects all routes including api/trpc routes
    // Please edit this to allow other routes to be public as needed.
    // See https://clerk.com/docs/references/nextjs/auth-middleware for more information about configuring your Middleware
    export default authMiddleware({
      publicRoutes: ['/site', '/api/uploadthing'],
      async beforeAuth(auth, req) {},
      async afterAuth(auth, req) {
        //rewrite for domains
        const url = req.nextUrl
        const searchParams = url.searchParams.toString()
        let hostname = req.headers

        const pathWithSearchParams = `${url.pathname}${
          searchParams.length > 0 ? `?${searchParams}` : ''
        }`

        //if subdomain exists
        const customSubDomain = hostname
          .get('host')
          ?.split(`${process.env.NEXT_PUBLIC_DOMAIN}`)
          .filter(Boolean)[0]

        if (customSubDomain) {
          return NextResponse.rewrite(
            new URL(`/${customSubDomain}${pathWithSearchParams}`, req.url)
          )
        }

        if (url.pathname === '/sign-in' || url.pathname === '/sign-up') {
          return NextResponse.redirect(new URL(`/agency/sign-in`, req.url))
        }

        if (
          url.pathname === '/' ||
          (url.pathname === '/site' && url.host === process.env.NEXT_PUBLIC_DOMAIN)
        ) {
          return NextResponse.rewrite(new URL('/site', req.url))
        }

        if (
          url.pathname.startsWith('/agency') ||
          url.pathname.startsWith('/subaccount')
        ) {
          return NextResponse.rewrite(new URL(`${pathWithSearchParams}`, req.url))
        }
      },
    })

    export const config = {
      matcher: ['/((?!.+\\.[\\w]+$|_next).*)', '/', '/(api|trpc)(.*)'],
    }
    ```

2. Create a page for our agency. Go to `src/app/agency` and create your `page.tsx` in it and get the basic structure.

    ```tsx
    // src/app/agency/page.tsx

    import React from 'react'

    const Page = () => {
        return <div>Agency Dashboard</div>
    }

    export default Page
    ```

3. Set up the logic of the 404 error in case the domain does not exist. Go to `src/app/[domain]/[path]/page.tsx`.

    ```tsx
    ????
    ```

## Prisma Installation

1. Search for “Use Bun for Prisma” : https://bun.sh/guides/ecosystem/prisma#:~:text=Prisma works out of the,initialize it with bun init .&text=Then install the Prisma CLI,prisma%2Fclient )%20as%20dependencies.&text=We'll%20use%20the%20Prisma,our%20schema%20and%20migration%20directory.

    ```bash
    bun add prisma @prisma/client
    bunx prisma init —datasource-provider mysql
    ```

2. After the installation, our `schema.prisma` will be created. We also need to add in `datasource db` our `relationMode` ( Line 11, just after `url = env` ), and add:
    ```bash
    relationMode= “prisma”
    ```

3. Go to your `lib` folder and create a `db.tsx` file to initialize our client:

    ```tsx
    import { PrismaClient } from '@prisma/client'

    declare global {
        var prisma: PrismaClient | undefined
    }

    export const db = globalThis.prisma || new PrismaClient()

    if (process.env.NODE_ENV !== 'production') globalThis.prisma = db
    ```

# Part 3

## Database Schema

We need to create a module for each of the references (Agency, Role, Permissions[], Notification[]). An example of a structured database schema is provided in the repository.

Once everything is set, we can run the following commands in the terminal to generate our database and push it:

```bash
bunx prisma generate
bunx prisma db push
```

To see the tables in the database, type bunx prisma studio in a new terminal. Cross-check the tables with the ones in Workbench.

Building Agency Section
We need to change our src/app/agency/page.tsx to redirect the user to the login page.
Create a new file in src/app/lib/queries.ts and set it up. We also need to implement a logic to check whether the user has an existing account, so if an invitation is sent we avoid the creation of a new user.
Time to implement agency/page.tsx. For us to send them to that agency we also need to check if they are a subaccount user. Optionally, incorporate a guest account functionality so the user can send to the client an invitation and check in the website.

``` typescript
import AgencyDetails from '@/components/forms/agency-details'
import { getAuthUserDetails, verifyAndAcceptInvitation } from '@/lib/queries'
import { currentUser } from '@clerk/nextjs'
import { Plan } from '@prisma/client'
import { redirect } from 'next/navigation'
import React from 'react'

const Page = async ({
  searchParams,
}: {
  searchParams: { plan: Plan; state: string; code: string }
}) => {
  const agencyId = await verifyAndAcceptInvitation()
  console.log(agencyId)

  //get the users details
  const user = await getAuthUserDetails()
  if (agencyId) {
    if (user?.role === 'SUBACCOUNT_GUEST' || user?.role === 'SUBACCOUNT_USER') {
      return redirect('/subaccount')
    } else if (user?.role === 'AGENCY_OWNER' || user?.role === 'AGENCY_ADMIN') {
      if (searchParams.plan) {
        return redirect(`/agency/${agencyId}/billing?plan=${searchParams.plan}`)
      }
      if (searchParams.state) {
        const statePath = searchParams.state.split('___')[0]
        const stateAgencyId = searchParams.state.split('___')[1]
        if (!stateAgencyId) return <div>Not authorized</div>
        return redirect(
          `/agency/${stateAgencyId}/${statePath}?code=${searchParams.code}`
        )
      } else return redirect(`/agency/${agencyId}`)
    } else {
      return <div>Not authorized</div>
    }
  }
  const authUser = await currentUser()
  return (
    <div className="flex justify-center items-center mt-4">
      <div className="max-w-[850px] border-[1px] p-4 rounded-xl">
        <h1 className="text-4xl"> Create An Agency</h1>
        <AgencyDetails
          data={{ companyEmail: authUser?.emailAddresses[0].emailAddress }}
        />
      </div>
    </div>
  )
} 

export default Page
```

Building Forms
Let’s create the <AgencyDetails> component. Go to src/components and create a “forms” folder with an “agency-details.tsx” file.

Going back to page.tsx (agency), we need to import the login verification. We will not need Line 7, 8 anymore because the function verifyAndAcceptInvitation() is already checking the credentials and redirecting to the sign-in page.

```
// Line 6

const Page = async () => {
	const agencyId = await verifyAndAcceptInvitation()
	console.log(agenctId)
}
```

Part 4
In progress…