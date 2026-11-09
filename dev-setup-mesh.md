# Setup Mesh

Complete this setup one time on each machine. It confirms that the `mesh` command works and that it finds the mesh root of this Flint. The Onyx login is optional.

# Prerequisites

- The `mesh` command is on the `PATH`. The Flint CLI installs it: `npm install -g @nuucognition/flint-cli`. On a development machine, ndv installs it: `ndv use mesh dev` or `ndv use mesh prod`.
- For Onyx work only: a NUU account that is a member of Onyx.

# Actions

1. **Check the command.**
   ```bash
   mesh --version
   ```
   If the shell cannot find `mesh`, stop. Ask the person to install the Flint CLI, or to use ndv on a development machine.

2. **Check the mesh root.** Run this command in the Flint root.
   ```bash
   mesh root
   ```
   The root must be the Flint root. If the command finds no root, the Flint has no `mesh.json`. Ask the person before you run `mesh init`.

3. **Check the mesh.**
   ```bash
   mesh info
   ```
   Record the document count and the diagnostic counts. A diagnostic does not stop the setup.

4. **Check the Onyx login (optional).** Do this step only when the machine will sync with Onyx.
   ```bash
   mesh onyx whoami
   ```
   Exit 0 shows the account. Exit 3 means that there is no valid login. Only a person can log in: `mesh onyx login` opens a browser, and it refuses to run without a terminal. Ask the person to run it in a terminal. Do not log in for the person.

5. **Mark setup complete.**
   ```bash
   flint shard setup mesh --complete
   ```

# Verification

- `mesh --version` prints a version.
- `mesh root` prints the Flint root.
- For Onyx work: `mesh onyx whoami` exits 0, and `mesh onyx list` lists the Onyx meshes of the account.
