I think this is a good simple example that demonstrates how we should probably implement a page  
[https://github.com/comvex-jp/digima-adminweb-app/pull/13](https://github.com/comvex-jp/digima-adminweb-app/pull/13)

1. Add GraphQL operation (`codegen/adminweb-bff/accounts/accounts.graphql`)
2. `bun run codege` -> generates GraphQL related code under `/generated/adminweb-bff`
3. Create reusable components under `components/` without any domain context (NO components under this directory should have domain/resource related words)
4. We could extract the resource query hook as a custom hook under `models/` but we don't need to, unless it's really going to be reused in other places (`const { data, loading, error } = useQuery<AccountQuery, AccountQueryVariables>(AccountDocument, {...`)
5. Add model specific functions under `models/` (`src/models/account/utils.ts`). I call it models because it's frontend specific models that are not necessarily the same as the domains in the backend
6. Add model-agnostic functions under `utils/` (`src/utils/time/index.ts`)
7. Finally assemble everything in the page component (`src/components/pages/accounts/AccountDetails.tsx`)

And I think you already realized, I only created `index.ts` under `src/utils/time/`. I think should split it into files/sub-directories when we have clear ideas after having more functions. The old web app had too many pre-defined structures, let's keep it as small as possible at the beginning and change laterI shared this a few times but I do think this is the crux of the approach that differentiates our new web apps from the old user web app  
[https://comvex.slack.com/archives/C09G9RSMVDH/p1764639407196349](https://comvex.slack.com/archives/C09G9RSMVDH/p1764639407196349)Even after I leave the team, do always talk to me if you have anything because I want to keep the current clean state of these web apps.