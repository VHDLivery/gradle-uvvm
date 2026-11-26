# gradle-uvvm

Packages the complete [UVVM](https://github.com/UVVM/UVVM) distribution as one
Maven source ZIP, including UVVM Utility Library, the VVC framework, the Bitvis
VIPs, and the upstream scripts, documentation and license files. The release
commit pins the UVVM revision through the Git submodule at `upstream/uvvm`.
The current pin is upstream [2026.10.27](https://github.com/UVVM/UVVM/tree/3823d3a45a1489b88dc0e16c790cf37c3de2efd1).

## Build locally

Initialize the submodule and publish to a file-based Maven repository.

```sh
git submodule update --init --recursive
./gradlew publishUvvmPublicationToBuildRepository -PpackageVersion=2026.10.27
```

The ZIP is in `build/distributions` and the Maven repository is in
`build/repository`. The version above is an example. Use a version that
identifies the UVVM revision you have selected.

## Publish a release

Update the submodule to the desired upstream revision and commit the gitlink.
Publish a GitHub Release from that commit with a tag such as `v2026.10.27`.
The release workflow checks out the pinned UVVM revision and publishes
`io.github.vhdlivery.uvvm:uvvm` to GitHub Packages. The Maven version is the
release tag without its leading `v`. After publishing the Maven package, the
workflow attaches `uvvm-<version>-src.zip` and `uvvm-<version>.pom` to the same
GitHub Release.

## Use with gradle-vhdlArtifacts

Add the Maven repository at
`https://maven.pkg.github.com/vhdlivery/gradle-uvvm` to your Gradle build with
GitHub Packages credentials. Then declare the simulation dependency with the
version of the published release.

```groovy
dependencies {
    sim 'io.github.vhdlivery.uvvm:uvvm:2026.10.27'
}
```

Run `./gradlew unzipSimArtifacts`. The plugin selects the `src` ZIP and strips
its outer `uvvm-<version>/` directory, leaving the upstream tree directly in
`build/dependency`. For example, the utility library lands at
`build/dependency/uvvm_util`, and the full upstream compile script is
`build/dependency/script/compile_all.sh`.

Gradle's default file filters omit `.gitattributes` and `.gitignore` when
extracting the ZIP. To extract these upstream files too, add this to the
consumer's `settings.gradle`.

```groovy
org.apache.tools.ant.DirectoryScanner.removeDefaultExclude('**/.gitattributes')
org.apache.tools.ant.DirectoryScanner.removeDefaultExclude('**/.gitignore')
```

In `amba-pkg_axi4s`, pass `--uvvm-root build/dependency` to the existing
`src/sim/scripts/run.py` VUnit runner. It reads UVVM Utility Library sources
in the order specified by `uvvm_util/script/compile_order.txt`.
