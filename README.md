name: Build release bundle

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Unzip project
        run: unzip -q DesiFarming.zip

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17

      - uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: 8.11.1

      - name: Decode keystore
        run: echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 -d > "$RUNNER_TEMP/upload.jks"

      - name: Build signed AAB and APK
        working-directory: DesiFarming
        env:
          KEYSTORE_PATH: ${{ runner.temp }}/upload.jks
          KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
        run: gradle :app:bundleRelease :app:assembleRelease --no-daemon

      - uses: actions/upload-artifact@v4
        with:
          name: DesiFarming-release
          path: |
            DesiFarming/app/build/outputs/bundle/release/*.aab
            DesiFarming/app/build/outputs/apk/release/*.apk
