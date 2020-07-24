## Repository Setup Guide

In this doc we will casually discuss ideas & hints for creating your project repository.

### Fullstack Repo or separate Repos

The central decision in your project setup will be:
- Do you use ONE Git repository for your frontend AND your backend? 
- Or do you use SEPARATE repositories for each?

Both approaches have their pros & cons.

Let's address them.

#### Separate Repos

We use separate Git repos for our Frontend & Backend app. 

##### Pro
- Easier to prevent working on same files & avoid merge conflicts
- Focus: Frontend / Backend teams can easily separate their work and get not disturbed by commits of the other team
- Typically in companies repos are separated. So we train the real life workflow here

##### Con
- Maintain two repos & two domains
- Configuration of connection between frontend & backend
- More testing effort
  - always need to pull from TWO repos and start TWO apps locally


#### One Repo for both ("Mono Repo")

We use just one repository having all Express and React files combined in just one folder structure.

##### Pro
- Easier to configure frontend backend connection (all on just one domain)
- Just one repository & domain to maintain

##### Con
- Whole team is working on same codebase: merge conflicts will be seen more often
  - also a pro: will train you to solve git conflicts :-)
- Performance: Serving every React file over Express causes a slight overhead on the frontend on first load
- Real Life relation: This workflow is almost never used in companies (due to team specialization)
  - Approach just suited for smaller projects (e.g. the final project)


#### Conclusion

So what approach to take?

It depends on your learning preference - what do you train?

- Do you wanna train Git heavily? 
  - Then the MonoRepo approach will give you way more opportunity. You probably will need to pull and merge much work of others more often and need to solve conflicts frequently

- Do you have a team where everybody works a bit on everything (frontend and backend)? 
  - Mono Repo will make it a bit easier in this case, because not everybody will need to maintain two repos constantly

- Do you put most value on training the real life workflow that you probably will face in a company and prepare yourself for that?
  - Then go for SEPARATE repositories
