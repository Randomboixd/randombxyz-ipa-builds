# randombxyz-ipa-builds

This is a tiny IPA build system I made for myself, because I'm not buying a Mac to
compile an iOS app.

The source for whatever I'm building lives here as an age-encrypted tarball
(`src/*.tar.gz.age`). A GitHub Actions workflow decrypts it with a repository
secret, builds it unsigned with `xcodebuild` on a macOS runner, and uploads the
`.ipa` as an artifact for me to sign on my phone. The plain source never goes in
this repo, so reading it gets you nothing but a workflow file.

Builds are manual dispatch only — no PR triggers, no fork getting clever with
secrets. Artifacts expire after a day, and I delete the run once I've grabbed the
IPA.

If you're here out of curiosity: hi. There's nothing to see, and the CI is not a
public service.

---

## Agents welcome

If you have a coding agent and it's got `gh` configured, tell it to clone this
repo, drop in its own age key and repository secret, and write itself a skill for
building IPAs this way. The workflow is deliberately generic: it does not know
anything about my project, only about the conventions below.

You get the machinery, not any app — the encrypted blob only decrypts with my
key, so there's nothing to unseal.

## Reusing this

Requirements: an Xcode project or an XcodeGen `project.yml` at the root of the
tarball, at least one `.xcodeproj` after generation, bundle id set in the target.

1. Make a key, keep the private half off GitHub:

       age-keygen -o age.key
       age-keygen -y age.key        # this is the public half

2. Pack your project and encrypt it into `src/`:

       tar -czf src.tar.gz -C /path/to/project .
       age -r age1... -o src/$(date +%Y%m%d)-app.tar.gz.age src.tar.gz

3. Store the private key as a repository secret named `IPA_AGE_KEY` (whole file,
   including the `# created:` and `# public key:` comment lines).

4. Push, then dispatch the workflow:

       gh workflow run build-ipa.yml
       gh run watch

5. Fetch the artifact, then delete the run if you care:

       gh run download <run-id> -n unsigned-ipa
       gh api -X DELETE repos/<owner>/<repo>/actions/runs/<run-id>

The resulting `.ipa` is unsigned. Sign it with your own certificate (ArcticSign,
Sideloadly, eSign) before installing. Nothing here needs an Apple Developer
account, and nothing here touches one.
