# Nix expressions for VS Code Extensions

At the time of writing this, `nixpkgs` contains **271** `VS Code` extensions. This is a small fraction of the more than **40,000** extensions in the `VS Code Marketplace`! In addition, many of the extensions in `nixpkgs` are significantly out-of-date.

This flake provides Nix expressions for the majority of available extensions from [Open VSX](https://open-vsx.org/) and [VS Code Marketplace](https://marketplace.visualstudio.com/vscode). A `GitHub Action` updates the extensions daily.

That said, you can now use a different set of extensions for `VS Code`/`VSCodium` in each of your projects. Moreover, you can share your flakes and cache them so that other people don't need to install these extensions manually!

## Note

- Check the NixOS wiki [page](https://wiki.nixos.org/wiki/Visual_Studio_Code) about VS Code.
- Check [nix4vscode](https://github.com/nix-community/nix4vscode) (and contribute!) if you need a more individual approach to extensions.
- Extension publishers and names are lowercased only in Nix.
  - They're not lowercased in `.json` cache files such as [data/cache/open-vsx-latest.json](./data/cache/open-vsx-latest.json).
- Access an extension in the format `<attrset>.<publisher>.<name>`, where `<attrset>` is `vscode-marketplace`, `open-vsx`, etc. (see [Explore](#explore)).
- If an extension publisher or name aren't valid Nix identifiers, quote them like `<attrset>."4"."2"`.
- We have a permission from MS to use a crawler on their API in this case (see the [discussion](https://github.com/NixOS/nixpkgs/issues/208456)). Please, don't abuse this flake!
- Some previously available extensions may be unavailable in newer versions of this flake.
  - An extension is missing if it doesn't appear during a particular workflow run in a `VS Code Marketplace` or an `Open VSX` response about the full set of available extensions ([discussion](https://github.com/nix-community/nix-vscode-extensions/issues/16#issuecomment-1441025955)).
  - We let missing extensions remain in cache files (see [data/cache](./data/cache)) at most `maxMissingTimes` (specified in [.github/config.yaml](.github/config.yaml)).
- We don't automatically handle extension packs. You should look up extensions in a pack and explicitly write all necessary extensions.
- Unfree ([wiki](https://wiki.nixos.org/wiki/Unfree_software)) extensions from `nixpkgs` stay unfree here (see [Special extensions](#special-extensions)). If you want to use unfree extensions, try one of the following ways:
  - Allow unfree packages ([manual](https://nixos.org/manual/nixpkgs/stable/#sec-allow-unfree)).
  - Update the license of a particular extension `(<publisher>.<name>.override { meta.license = [ ]; })`.

## Template

This repository has a flake [template](template/flake.nix).

This template provides a [VSCodium](https://github.com/VSCodium/vscodium) with a couple of extensions.

1. Create a flake from the template (see [nix flake new](https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-flake-new.html)).

   ```console
   nix flake new vscodium-project -t github:nix-community/nix-vscode-extensions
   cd vscodium-project
   git init && git add .
   ```

1. Run `VSCodium`.

   ```console
   nix run .# .
   ```

1. Alternatively, start a devShell and run `VSCodium`. A `shellHook` will print extensions available in the `VSCodium`.

   ```console
   nix develop
   codium .
   ```

In case of problems see [Troubleshooting](#troubleshooting).

## Example

[flake.nix](./flake.nix) provides a default package using [vscode-with-extensions](https://github.com/NixOS/nixpkgs/blob/81b9a5f9d1f7f87619df26a4eaf48bf6dec8c82c/pkgs/applications/editors/vscode/with-extensions.nix) from `nixpkgs`.

This package is `VSCodium` with a couple of extensions.

Run `VSCodium` and list installed extensions.

```console
nix run github:nix-community/nix-vscode-extensions# -- --list-extensions
```

## Usage

> [!NOTE]
> Check the NixOS wiki [page](https://wiki.nixos.org/wiki/Visual_Studio_Code) about VS Code.

### Extensions

We provide extensions attrsets that contain both universal and platform-specific extensions.
We use a reasonable mapping between the sites target platforms and Nix-supported platforms (see the [issue](https://github.com/nix-community/nix-vscode-extensions/issues/20) and `systemPlatform` in [flake.nix](./flake.nix)).

There are several attrsets:

- `vscode-marketplace` and `open-vsx` contain the latest versions of extensions, including pre-release ones. Such pre-release versions expire in some time. That's why, there are `-release` attrsets.
- `vscode-marketplace-release` and `open-vsx-release` contain the release versions of extensions (see [Release extensions](#release-extensions)).
- `forVSCodeVersion "4.228.1"` allows to leave only the extensions [compatible](https://code.visualstudio.com/api/working-with-extensions/publishing-extension#visual-studio-code-compatibility) with the `"4.228.1"` version of `VS Code`.
  - You may supply the actual version of your `VS Code` instead of `"4.228.1"`.

> [!NOTE]
> In `with A; with B;`, the attributes of `B` shadow the attributes of `A`.
> Keep in mind this property of `with` when writing `with vscode-marketplace; with vscode-marketplace-release;`.
> See [With-expressions](https://nix.dev/manual/nix/latest/language/syntax#with-expressions).

### With flakes

See [Flakes](https://wiki.nixos.org/wiki/Flakes).

#### Overlay

See [Overlays](https://wiki.nixos.org/wiki/Overlays#Using_overlays).

If you use NixOS, Home Manager, or similar:

1. Add `nix-vscode-extensions` to the flake inputs ([example](https://github.com/maurerf/nix-darwin-config/blob/0f88b77e712f14e3da72ec0b640e206a37da7afe/flake.nix#L16)).

1. (Optional) Allow unfree packages ([example](https://github.com/maurerf/nix-darwin-config/blob/0f88b77e712f14e3da72ec0b640e206a37da7afe/flake.nix#L45)).

   - See [Note](#note) for other ways to use unfree extensions.

1. Add the default overlay from the `nix-vscode-extensions` flake to `nixpkgs.overlays` ([example](https://github.com/maurerf/nix-darwin-config/blob/0f88b77e712f14e3da72ec0b640e206a37da7afe/flake.nix#L48)).

1. Get extensions from `pkgs.vscode-marketplace` and/or `pkgs.open-vsx` ([example](https://github.com/maurerf/nix-darwin-config/blob/0f88b77e712f14e3da72ec0b640e206a37da7afe/flake.nix#L131)).

#### Standalone flake

See [Template](#template).

### Without flakes

> [!NOTE]
> The values of `url`, `ref`, `rev` and in the `fetchGit` argument are for demonstration purposes.
> The value `9edbf5d1c9c9b5c5dd1fa6d6fc0c3cd01ec09346` is the full SHA-256 hash of a commit in this repository.
> Replace it with the hash of the commit you need.

```nix
let
  system = builtins.currentSystem;
  extensions =
    (import (builtins.fetchGit {
      url = "https://github.com/nix-community/nix-vscode-extensions";
      ref = "refs/heads/master";
      rev = "9edbf5d1c9c9b5c5dd1fa6d6fc0c3cd01ec09346";
    })).extensions.${system};
  extensionsList = with extensions.vscode-marketplace; [
      rust-lang.rust-analyzer
  ];
in ...
```

## History

You can search for an extension in the repository history:

- Get commits containing the extension: `git log -S '"copilot"' --oneline data/cache/vscode-marketplace-latest.json`
- Select a commit, e.g.: `0910d1e`
- Search in that commit: `git grep '"copilot"' 0910d1e -- data/cache/vscode-marketplace-latest.json`

## Explore

Explore extensions via `nix repl`.

> [!NOTE]
> Press the `Tab` button (denoted as `<TAB>` below) to see attrset attributes.

### Get your system

```console
nix-instantiate --eval --expr 'builtins.currentSystem'
```

Output on my machine:

```console
"x86_64-linux"
```

### Get the `extensions` attrset

#### Get `extensions` with flakes

```console
$ nix repl

nix-repl> :lf github:nix-community/nix-vscode-extensions/9edbf5d1c9c9b5c5dd1fa6d6fc0c3cd01ec09346
Added 10 variables.

nix-repl> t = extensions.<TAB>
extensions.aarch64-darwin  extensions.aarch64-linux   extensions.x86_64-darwin   extensions.x86_64-linux

nix-repl> t = extensions.x86_64-linux

nix-repl> t.<TAB>
t.forVSCodeVersion            t.open-vsx-release            t.vscode-marketplace-release
t.open-vsx                    t.vscode-marketplace
```

#### Get `extensions` without flakes

> [!NOTE]
> The values of `url`, `ref`, `rev` and in the `fetchGit` argument are for demonstration purposes.
> The value `9edbf5d1c9c9b5c5dd1fa6d6fc0c3cd01ec09346` is the full SHA-256 hash of a commit in this repository.
> Replace it with the hash of the commit you need.

```console
$ nix repl

nix-repl> t1 = (import (builtins.fetchGit {
                url = "https://github.com/nix-community/nix-vscode-extensions";
                ref = "refs/heads/master";
                rev = "9edbf5d1c9c9b5c5dd1fa6d6fc0c3cd01ec09346";
              }))

nix-repl> t = t1.extensions.<TAB>
t1.extensions.aarch64-darwin  t1.extensions.aarch64-linux   t1.extensions.x86_64-darwin   t1.extensions.x86_64-linux

nix-repl> t = t1.extensions.x86_64-linux

nix-repl> t.<TAB>
t.forVSCodeVersion            t.open-vsx-release            t.vscode-marketplace-release
t.open-vsx                    t.vscode-marketplace
```

### Pre-release versions

```console
nix-repl> t.vscode-marketplace.rust-lang.rust-analyzer
«derivation /nix/store/58am52cg9hbiailq5jaycmfbibgcsck1-vscode-extension-rust-lang-rust-analyzer-0.4.2324.drv»
```

### Release versions

```console
nix-repl> t.vscode-marketplace-release.rust-lang.rust-analyzer
«derivation /nix/store/1l2q4iy939n975cmwnzg44dbhwkb2509-vscode-extension-rust-lang-rust-analyzer-0.3.2319.drv»
```

### Pre-release versions compatible with a given version of VS Code

```console
nix-repl> (t.forVSCodeVersion "1.78.2").vscode-marketplace.rust-lang.rust-analyzer
«derivation /nix/store/iag019w3v7jbypj9d6qz03bh0xmaf248-vscode-extension-rust-lang-rust-analyzer-0.4.1067.drv»
```

### Removed extensions

Some extensions are hard to handle correctly ([example](https://github.com/nix-community/nix-vscode-extensions/issues/69)), so they have been removed via [removed.nix](./removed.nix).

They may be available in `nixpkgs`, in `pkgs.vscode-extensions` ([link](https://search.nixos.org/packages?channel=unstable&from=0&size=50&sort=relevance&type=packages&query=vscode-extensions)).

### Apply the overlay

See [Overlay](#overlay).

> [!NOTE]
> The value `9edbf5d1c9c9b5c5dd1fa6d6fc0c3cd01ec09346` is the full SHA-256 hash of a commit in this repository.
> Replace it with the hash of the commit you need.

#### Apply the overlay with flakes

```console
nix-repl> :lf github:nix-community/nix-vscode-extensions/9edbf5d1c9c9b5c5dd1fa6d6fc0c3cd01ec09346
Added 14 variables.

nix-repl> extensions = import inputs.nixpkgs { system = builtins.currentSystem; config.allowUnfree = true; overlays = [ overlays.default ]; }
```

#### Apply the overlay without flakes

```console
nix-repl> nix-vscode-extensions = (import (builtins.fetchGit {
                url = "https://github.com/nix-community/nix-vscode-extensions";
                ref = "refs/heads/master";
                rev = "9edbf5d1c9c9b5c5dd1fa6d6fc0c3cd01ec09346";
              }))

nix-repl> extensions = import <nixpkgs> { system = builtins.currentSystem; config.allowUnfree = true; overlays = [ nix-vscode-extensions.overlays.default ]; }
```

#### Get an unfree extension

```console
nix-repl> extensions.vscode-marketplace.ms-python.vscode-pylance
«derivation /nix/store/dn9kklr6vq8qfmq2bp32l3av4n5500li-vscode-extension-ms-python-vscode-pylance-2025.2.102.drv»
```

## Contribute

### Issues

Resolve [issues](https://github.com/nix-community/nix-vscode-extensions/issues).

### README

- Fix links.
- Write new sections.
- Update commit SHA used in examples if they're too old.
- Enhance text.

### Release extensions

The [config](.github/config.yaml) contains several extensions.
We cache the information about the latest **release** versions of these extensions (see [Extensions](#extensions)).

You can add new extensions to the config and make a Pull Request.
Use the original extension publisher and name, e.g. `GitHub` and `copilot`.

### Extra extensions

The [extra-extensions.toml](extra-extensions.toml) file contains a list of extensions to be fetched from sites other than `VS Code Marketplace` and `Open VSX`.
These extensions replace ones fetched from `VS Code Marketplace` and `Open VSX`.
Add necessary extensions there, preferrably, for all supported platforms (see [Extensions](#extensions)).
[nvfetcher](https://github.com/berberman/nvfetcher) will fetch the latest release versions of these extensions and write configs to [generated.nix](data/extra-extensions/generated.nix).

### Special extensions

Certain extensions require special treatment.

Provide functions to build such extension in [mkExtension.nix](mkExtension.nix).

Optionally, create and link there issues explaining chosen functions.

Each extension, including [Extra extensions](#extra-extensions), is built via one of the provided functions.

These functions don't modify the license of ([unfree](https://wiki.nixos.org/wiki/Unfree_software)) extensions from `nixpkgs`.

#### Build problems

- Extension with multiple extensions in a zipfile ([issue](https://github.com/nix-community/nix-vscode-extensions/issues/31))
- Platform-specific extensions ([comment](https://github.com/nix-community/nix-vscode-extensions/issues/20#issuecomment-1543679655))

### Main flake

1. (Optionally) Install [direnv](https://direnv.net/), e.g., via `nix profile install nixpkgs#direnv`.

1. Run a devshell. When prompted about `extra-trusted-substituters` answer `y`. This is to use binary caches.

   ```console
   nix develop nix-dev/
   ```

1. (Optionally) Start `VSCodium` with necessary extensions and tools.

   ```console
   nix run nix-dev/#writeSettings
   nix run nix-dev/#codium .
   ```

### Haskell script

1. See the [README](./haskell/README.md).

1. Set the environment.

   ```console
   set -a
   source .env
   ```

1. Run the script.

   ```console
   nix run haskell/#updateExtensions
   ```

### Pull requests

Pull requests are welcome!

## Troubleshooting

- If `Nix`-provided `VSCodium` doesn't pick up the extensions:
  - Close other instance of `Nix`-provided `VSCodium`.
  - Try to reboot your computer and start `VSCodium` again.
- See [troubleshooting](https://github.com/deemp/flakes/blob/main/README/Troubleshooting.md).
