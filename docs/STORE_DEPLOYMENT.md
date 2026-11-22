# Store deployment guide

Store publishing is **opt-in** and separate from regular pull-request testing. Do not grant store credentials to untrusted pull requests.

## Android: Google Play Internal Testing

A reusable upload workflow is available at `.github/workflows/google-play-internal.yml`. It accepts a signed AAB artifact, an Android package name, an artifact-producing run ID, and a Base64-encoded Google Play service-account JSON secret.

Prerequisites:
1. Set up the app in Play Console and meet its initial release/API access prerequisites.
2. Enable Google Play Android Developer API in the linked Google Cloud project; configure a service account with minimum necessary app permissions in Play Console.
3. Add `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON_BASE64` as an Actions secret **in the game repository**. Never commit the JSON.
4. Configure Android upload signing secrets and ensure the AAB has a new versionCode.
5. Set up the `google-play-internal` environment in the game repository with required reviewers for human approval.
6. Use a trusted branch, manual dispatch, or protected release workflow to call the upload workflow. Provide `run_id: ${{ github.run_id }}` if the build artifact was generated earlier in the same workflow run.

The workflow intentionally only allows the `internal` track. It is not connected to every push by default. It has not yet been tested against a live Play Console app.

Example call (as a downstream job in the same game repository workflow, after a signed build job):

```yaml
  upload_internal:
    needs: unity
    if: github.event_name == 'workflow_dispatch'
    permissions:
      contents: read
      actions: read
    uses: eFurkanSngl/SGL-Unity-CI-CD/.github/workflows/google-play-internal.yml@main
    with:
      package_name: com.example.mygame
      artifact_name: android-MyGame
      run_id: ${{ github.run_id }}
    secrets: inherit
```

**Note:** GitHub reusable workflow jobs with `uses` cannot themselves specify an `environment`. Approval protection belongs to the called job's environment; ensure the `google-play-internal` environment and its required reviewers are configured in the calling game repository. Confirm approval behavior before production use. Pin to a reviewed commit SHA after validation.

## iOS: TestFlight (planned)

The iOS implementation is not yet included. It will require:
- macOS runner and Xcode
- Unity iOS build/export and compatible Unity licensing
- Apple Developer Program membership, app identifiers and provisioning/signing assets
- App Store Connect API key stored in game-repository secrets
- IPA export and Fastlane `pilot` upload to TestFlight

Do not upload Apple certificates, private keys, provisioning profiles, or API credentials to this public template. Apple review and external tester requirements may still apply.

## Security

- Keep secrets in the consuming game's repository, not this public workflow repository.
- Use least-privilege store API accounts and protected environments.
- Never print credentials in logs or include them in build artifacts.
- Do not expose signing or store secrets to pull requests from forks.
- Use immutable third-party action SHAs and versioned reusable workflow references for production.
