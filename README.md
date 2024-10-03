# project

This is the primary project repo.

## more info will go here, eventually

### submodules

To clone project:
`git clone --recurse-submodules <this repo's url>`

To update all submodules:
- in root repo (this one): `git pull`
- `git submodule update --init --remote --merge`

To update your submodule in root repo:
in root dir:
- `git add <submod name>` (eg `git add interface`)
- `git commit -m "updating <interface name>`

(JetBrains IDEs (IntelliJ, PyCharm, GoLand, etc) make this really, really easy!!)

Treat submodules like their own repos (they literally are!). Pull/fetch/etc inside a submodule will pull from *only* the submodule's repo, not the entire project.
  
