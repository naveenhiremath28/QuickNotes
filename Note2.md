```
BEGIN;

DELETE FROM proofs WHERE proof_profile = 'merkle-tree';

UPDATE transactions
SET proof_id = NULL,
    proof_status = 'pending',
    proof_profile = NULL,
    batch_id = NULL          -- ⚠️ see caveat below
WHERE proof_status = 'proven';

UPDATE token_transactions
SET proof_id = NULL
WHERE proof_id IS NOT NULL;

COMMIT;

```

data compliance
create incoming
