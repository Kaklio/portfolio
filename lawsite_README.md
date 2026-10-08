# Requirements:

###  1. Download and Install Node js
###  2. Download and MongoDB
###  3. Get the Official MongoDB and Tailwind Extensions in VScode
---------------

#### Run this command in an empty directory (This will be the root folder/directory)

```
npx create-next-app@latest
```

#### These Options will appear (Pick the Highlighted option)

1. Would you like to use TypeScript? **No** / Yes
2. Would you like to use ESLint? No / **Yes**
3. Would you like to use Tailwind CSS? No / **Yes**
4. Would you like your code inside a `src/` directory? **No** / Yes
5. Would you like to use App Router? (recommended) No / **Yes**
6. Would you like to use Turbopack for `next dev`?  No / **Yes**
7. Would you like to customize the import alias (`@/*` by default)? **No** / Yes
8. What import alias would you like configured? @/* **No** / Yes

---------------

#### Now Clone this Repository into the root directory, Run
```
git clone https://github.com/Kaklio/lawSite
```
Make sure the files cloned from the repository replace all local duplicates

#### Now Run (package.json json must be in root folder)
```
npm install
```
This will install all required packages from node package manager

Now Create a  ` .env.local` file in the root folder, it should have:
```
EMAIL_USER=YourEmail
EMAIL_PASS=GoogleAppPassword 

MONGODB_URI=mongodb://localhost:27017/Accounts

JWT_SECRET=OnlineGeneratedJWT

NEXTAUTH_SECRET=3j2+z8hQ9F1l2MxOyG5KX4QHzXf/YUfnQhPTzPxYDAk=
```

*Replace "YourEmail" with your email*

*Replace "GoogleAppPassword" with the google app password for the SAME email*

*(Search "app password" in your google account home page to generate one)*

*Generate and paste a JWT_SECRET online and paste in place of "OnlineGeneratedJWT"*

*For "NEXTAUTH_SECRET" use:*
`openssl rand -base64 32` *command to get a random 32-character string*

#### Finally Run:
```
npm run dev
```
#### Now Ctrl + Click on http://localhost:3000 or whatever link served to open the site 