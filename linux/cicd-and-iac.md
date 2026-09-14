# CI/CD and IaC (GitHub Actions + Terraform backend)

## What is it?

Patterns for scripting infrastructure setup steps (like creating an S3 remote
backend bucket) inside a GitHub Actions workflow's `run:` block.

## Why do we need it?

Terraform (and other IaC tools) need a remote backend bucket to exist *before*
`terraform init` can store state there. CI pipelines need to create that bucket
idempotently — safe to re-run without erroring if it already exists — and often
need to do this for more than one bucket (e.g. one per environment: dev,
staging, prod).

## Mental Model

> `if aws s3api head-bucket` = "does this exist already?" — check before you create, so re-running the pipeline doesn't fail or duplicate work.

> `for BUCKET_NAME in $BUCKET_NAMES; do ... done` = "repeat the same check-and-create for every name in the list."

## How it works

- `aws s3api head-bucket --bucket NAME` returns exit code 0 if the bucket
  exists, non-zero otherwise — that's why it works directly as an `if` condition.
- `2>/dev/null` suppresses the "not found" error noise when the bucket doesn't exist yet.
- Looping over `$BUCKET_NAMES` (a space-separated string) applies the same
  existence-check-then-create logic to multiple buckets without duplicating the block.

## Example

**Single bucket (original):**
```yaml
- name: Create remote backend bucket if not exists
  working-directory: IaC
  run: |
    if aws s3api head-bucket --bucket "$BUCKET_NAME" 2>/dev/null; then
      echo "Bucket $BUCKET_NAME already exists, skipping creation."
    else
      echo "Bucket $BUCKET_NAME does not exist, creating..."
      aws s3 mb "s3://$BUCKET_NAME" --region "$AWS_REGION"
    fi
```

**Looped over multiple buckets:**
```yaml
- name: Create remote backend buckets if not exists
  working-directory: IaC
  run: |
    for BUCKET_NAME in $BUCKET_NAMES; do
      if aws s3api head-bucket --bucket "$BUCKET_NAME" 2>/dev/null; then
        echo "Bucket $BUCKET_NAME already exists, skipping creation."
      else
        echo "Bucket $BUCKET_NAME does not exist, creating..."
        aws s3 mb "s3://$BUCKET_NAME" --region "$AWS_REGION"
      fi
    done
```

With the list defined alongside the region:
```yaml
env:
  BUCKET_NAMES: "my-tfstate-dev my-tfstate-staging my-tfstate-prod"
  AWS_REGION: us-east-1
```

## Common Commands

### `aws s3api head-bucket --bucket NAME`

Checks whether a bucket exists (and that you have access to it) — returns exit
code 0/non-0, no output on success. That's why it slots directly into an `if`.

### `aws s3 mb s3://NAME --region REGION`

**Flags:**
- `--region` → which AWS region to create the bucket in (required for anything outside `us-east-1` on many account setups)

`mb` = **m**ake **b**ucket.

## Common Mistakes

- Using a bash **array** (`BUCKET_NAMES=(a b c)`) and expecting it to survive
  through a GitHub Actions `env:` var — env vars are plain strings, arrays don't
  transfer. A space-separated string + word-splitting `for` loop is the simple
  fix, since S3 bucket names can't contain spaces anyway.
- Forgetting `2>/dev/null` on `head-bucket` — without it, every "doesn't exist
  yet" case prints a scary-looking AWS error even though the script handles it fine.
- Not quoting `"$BUCKET_NAME"` — breaks if a name ever contains something
  word-splitting-unsafe (defensive habit even though bucket names are usually safe).

## Interview Questions

**Q: How do you make an infrastructure-creation step in CI idempotent?**
> Check for existence first (e.g. `head-bucket`), and only create if the check
> fails. Re-running the pipeline then becomes a no-op instead of an error.

**Q: Why loop with a space-separated string instead of a bash array here?**
> GitHub Actions `env:` vars are strings, not arrays — a space-separated list
> with a plain `for x in $VAR` loop is the simplest thing that survives that
> boundary, and it's safe here because bucket names can't contain spaces.

**Q: What does `aws s3api head-bucket` actually check?**
> Whether the bucket exists **and** whether the caller has permission to access
> it — a 403 and a 404 both cause the command to fail, so it doubles as an
> access check.

## Interview Answer

> "I'd check for the bucket's existence with `aws s3api head-bucket` before
> creating it, so the step is idempotent on reruns. To handle multiple buckets,
> I loop over a space-separated list of names rather than a bash array, since
> GitHub Actions env vars don't preserve arrays — each name gets the same
> check-then-create logic inside the loop."

## Related Concepts

- [[linux/users-and-permissions]]
- [[linux/shell-basics]]

## Remember

```
head-bucket        → does it exist? (exit code, no output)
mb                  → make bucket
for x in $LIST      → repeat block per space-separated item
env vars in GH Actions → strings only, no native arrays
```
