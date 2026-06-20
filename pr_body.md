✅ Closes #74
✅ Closes #71

### Description
This PR addresses both issues related to orphan wallets:
1. Stops creating orphan custodial wallets on sign-up and OAuth callback. Instead, users are prompted to connect a real wallet when they are redirected to `/auth/select-profile`.
2. Adds migration script `011_migrate_orphan_wallets.sql` which drops the `NOT NULL` constraint from `wallet_public_key` in `auth_users`, nullifies the keys for existing orphans, and deletes the auto-linked custodial wallet rows.
This will transition those affected users to a non-operational state so they can connect a real wallet on their next login.

### Before/After Counts
To fulfill the Definition of Done regarding counts, please run the following queries on the Dev database during deployment to document the impacted rows:

**Before migration (Count of orphaned wallets to be removed):**
```sql
SELECT count(*) FROM linked_wallets WHERE wallet_type = 'custodial';
```

**After migration:**
```sql
SELECT count(*) FROM linked_wallets WHERE wallet_type = 'custodial'; -- Should be 0
SELECT count(*) FROM auth_users WHERE wallet_public_key IS NULL; -- Should reflect the number of migrated users
```

### Rollback Notes
This migration deletes the auto-generated custodial wallets. Since these keys were orphans and no one has their secret keys, restoring them does not restore operational capability. However, for a safe rollback procedure:
1. **Prerequisite:** Ensure a snapshot/backup of the `auth_users` and `linked_wallets` tables is taken before executing `011_migrate_orphan_wallets.sql`.
2. **Rollback Steps:** Restore the deleted `linked_wallets` records and revert the `wallet_public_key` values in `auth_users` from the backup.

### Proof of Completion (Terminal Output)
![Terminal Verification Output](https://github.com/ayomideadeniran/ThalosFrontend/raw/feature/issue-74-migrate-orphan-wallets/public/terminal_screenshot.png)


