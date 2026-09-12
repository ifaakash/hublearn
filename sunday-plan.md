# Sunday plan — fast, build-first

Where Saturday left off: `inventory.ini` has `moo` (Pi5) + `vm` (EC2, root
SSH), `ping-and-user-playbook.yml` does ping + user + file, `lab_user` is a
var, `--limit vm` already used once. Goal for today: max hands-on building,
hygiene as fast checkboxes only, pull weekend-2's role structure forward.

Write the YAML myself — this file is the checklist, not the code.

## 1. Two fast checkboxes (don't let this eat the morning)
- [ ] AWS Console → Billing → Budgets → $5 alarm.
- [ ] `vm`: `ansible_user` root → `ubuntu`/`ec2-user`, add `become: true` on
      the user-creation play (same as `moo`).
- [ ] (later today if there's a lull, not now) IAM daily-use user, root MFA.

## 2. Idempotence bug hunt (10 min)
- [ ] Predict: `file: state: touch` stamps mtime every run → `changed=1`
      forever. Confirm by re-running `--limit vm` unchanged.
- [ ] Fix the task so a second run reports `changed=0`.
- [ ] Re-run once more to prove it.

## 3. Break it, fast (15 min)
- [ ] Cause 2 breaks (bad path / flip `become` / typo module / bad indent).
- [ ] Log each in `docs/runbook.md`, 4 lines only:
      `Symptom / Task / Layer / Fix`.

## 4. Main event — turn this into a role (30-40 min)
- [ ] `roles/labuser/tasks/{main,provision,revoke}.yml`.
- [ ] `jit_action: provision|revoke` extra_var; `revoke` sets the user
      `state: absent`.
- [ ] Move `owner: 1006` into a var alongside `lab_user` (finish the
      refactor).
- [ ] Prove the round-trip:
      `ansible-playbook site.yml --limit vm -e jit_action=provision`
      → `id ubuntu` on `vm` (user exists)
      `ansible-playbook site.yml --limit vm -e jit_action=revoke`
      → `id ubuntu` on `vm` (fails)

## 5. Read the appendix (5 min)
- [ ] Skim `weekend-01-bootstrap-lab.md` appendix `site.yml`. Predict its
      recap in your head. Don't run it — just compare shape to mine.

## 6. Network one-liner (5 min)
- [ ] `dig`, `curl -vI`, `ss -lntp` once against `vm`. One line each in this
      file's notes on why `moo` (Tailscale) and `vm` (AWS hostname) resolve
      differently.

## 7. Demo + commit (5 min)
- [ ] Fresh terminal: provision → id → revoke → id (fails) on `vm`. Narrate
      inventory / module / idempotence / extra_vars out loud.
- [ ] One commit: `git add roles docs && git commit -m "Weekend 1: role +
      JIT provision/revoke round-trip"`.

## Skip today (do this week, don't gate building)
IAM user, MFA, `.gitignore` hardening (`*.pem`, `id_rsa`, `.aws/credentials`),
git author identity cleanup (commits currently come from 3 different
identities), an ADR write-up of the hygiene-after-EC2 ordering.

## Carries into weekend 2
Already ahead: `--limit` + a role exist before weekend 2 starts. Next
Saturday starts at "second role / `cloud` group with more than one EC2 host"
and `--limit` across *groups*, not from zero.
