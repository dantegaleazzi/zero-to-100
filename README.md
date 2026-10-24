# Zero to 100

I'm [Dante Galeazzi](https://dantegaleazzi.com). I build products solo with AI and share everything I learn and use along the way.

This is where I collect the skills, workflows, tools and open-source projects that come out of that process.

Want the next skill, experiment or write-up? **[Sign up for the newsletter →](https://dantegaleazzi.com)**

## Open-source projects

### [Motion Lab](https://github.com/dantegaleazzi/motion-lab)

An image-animation playground for your next website. Bring your images, customize the motion and take the code. It started as an animation for one of my products, then became a project anyone can use.

[Try the playground →](https://dantegaleazzi.github.io/motion-lab/) · [Get the code →](https://github.com/dantegaleazzi/motion-lab)

## Skills and guides

### Databases

#### Supabase → Neon

I decided to move one of my projects from Supabase to Neon. Here's a skill to help you work through that move, including the parts a database dump won't cover: Auth, permissions, files, background jobs and recovery.

| Resource | What you'll find |
| --- | --- |
| [Migration skill](skills/supabase-to-neon/SKILL.md) | Instructions for your AI agent to audit the app, plan the move and verify the result. |
| [Step-by-step guide](guides/supabase-to-neon.md) | What changes, what to keep, how to rehearse and what to check before switching. |
| [Migration plan template](templates/supabase-to-neon-plan.md) | A place to record dependencies, costs, acceptance checks and recovery steps. |

Start with the audit. Moving the database and replacing the entire Supabase backend are different scopes.

## Use a skill with your AI

Give your coding assistant the skill file and the path to your application. For example:

```text
Read skills/supabase-to-neon/SKILL.md.
Audit my app at <application path> in read-only mode.
Map its Supabase dependencies and prepare a migration plan using
templates/supabase-to-neon-plan.md.
Keep this first pass to analysis; don't change production or run migrations.
```

More skills, guides and projects as I build. Follow along at [dantegaleazzi.com](https://dantegaleazzi.com).
