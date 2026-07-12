---
title: "Using yq to Edit and Mutate JSON Files"
---

_Written with the help of Gemini_

## Using yq to Edit and Mutate JSON Files

When it came to managing configuration files, I was looking for a clean way to modify large JSON files—specifically to alter certain values dynamically based on the deployment environment.
A colleague had previously written a solution using `jq`, but mutating values and adding nodes resulted in quite complex Bash scripts and intermediate text files to describe the changes.
It felt messy, and I searched for a better way.

That is when I stumbled upon **yq** (specifically Mike Farah’s version).
While `jq` is the undisputed king of querying and filtering, `yq` was designed from the ground up for **modifying** and manipulating data structures (including JSON, YAML, TOML, and XML).

👉 Check out the project on GitHub: [mikefarah/yq](https://github.com/mikefarah/yq)

Every operation I needed is natively supported: changing values, adding or deleting nodes, and moving elements around the JSON tree.
The real superpower of this tool is the ability to chain all your mutations together into a single expression file (usually named `*.yq`).
As an added bonus, AI tools (like Claude, Gemini or ChatGPT) understand this syntax perfectly; if you describe the transformation you want on a simple case, they can generate the `yq` script for you flawlessly.

To prevent copy-paste mistakes when reordering lines later, a great best practice is to use the identity operator (`.`) on the first line.
This allows you to cleanly prefix every single actual change with a pipe (`|`) uniformly.

Here is what a mutation file `my-changes.yq` file looks like:

```jq
# ==========================================
# yq mutations applied to environment 1
# ==========================================
.

# Remove an unwanted info node
| del(.info)

# Inject a version marker into the metadata using a CI environment variable
| .root.metadata.import-version = "v_" + strenv(CI_COMMIT_SHORT_SHA)

# Modify configuration strategies
| .hints.strategy = "KEEP"
| .hints.custom = "DELETE" 

```

To run this in your CI/CD pipeline or locally without modifying your original source file, you just execute a single command:

```bash
yq --from-file my-changes.yq input.json > output.json
```

### Real-World File Structure

We use this approach to generate environment-specific configurations from a baseline JSON file.

```text
├── input.json                # The baseline input file
└── adjustments/
    ├── global/
    │   └── changes.yq        # Common mutations applied to all environments
    └── envs/
        ├── env1/
        │   └── changes.yq    # Mutations specific to Env 1
        └── env2/
            └── changes.yq    # Mutations specific to Env 2

```

Commands:

**Test only the global changes:**

```bash
yq -o=json --from-file adjustments/global/changes.yq input.json > out/output.json
```

**Create the configuration file for env1:**

```bash
yq -o=json --from-file adjustments/global/changes.yq input.json | yq -o=json --from-file adjustments/envs/env1/changes.yq > out/env1.json
```

**Create the configuration file for env2:**

```bash
yq -o=json --from-file adjustments/global/changes.yq input.json | yq -o=json --from-file adjustments/envs/env2/changes.yq > out/env2.json
```

This layout allows you to maintain a pristine master JSON file while isolating environment-specific variances. No more complex Bash wrappers, just clean, pipeline-friendly streaming configurations!
