---
name: deploy
description: Trigger deployments for ionstudioapps projects. Use when asked to deploy, build, or ship — wheeltodo mobile (EAS) or web (Vercel).
---

# ionstudioapps Deploy

Determine what to deploy based on context, then run the appropriate command.

## wheeltodo — Mobile (EAS Build)

Working directory: `apps/mobile` inside the wheeltodo repo.

| Profile | Command | Use for |
|---------|---------|---------|
| preview | `eas build --platform all --profile preview` | Testing builds (APK + IPA) |
| android only | `eas build --platform android --profile preview` | Android testing |
| ios only | `eas build --platform ios --profile preview` | iOS/TestFlight testing |
| production | `eas build --platform all --profile production` | App store releases |

After a build completes, share the EAS build URL with the team.

## wheeltodo — Web (Vercel)

The web app auto-deploys on push to `main`. To trigger manually:
```bash
cd apps/web && vercel --prod
```

Preview deployments happen automatically on every branch push.

## Steps

1. Ask which project and environment if not clear from context
2. Confirm the target before running (production deploys need explicit confirmation)
3. Run the build command from the correct directory
4. Report the build URL or deployment URL when done

## Notes
- EAS builds run in the cloud — no local Android/iOS SDK needed
- iOS builds require the EAS account to be logged in (`eas whoami`)
- The EAS project owner is `ionstudio`, project ID: `58443556-af7a-48c3-adda-b6db805ec166`
