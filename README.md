# Solana Mainnet-Beta Cluster Restart 6 February 2024

You need the following shred version and expected-bank-hash to join the cluster:

PENDING


## Identify your highest optimistic slot:

Highest optimistically confirmed slot: 246464040
^ Find this by grepping your logs for “optimistic_slot”:


grep "optimistic_slot slot" /path/to/log/file | tail

The highest slot might not be the last to appear in your logs, identify the highest of those printed.

Note: If your last confirmed slot is lower than that the one listed above, this is likely your node crashed before it was able to observe the latest supermajority. In this case, update according to step 2 then proceed to the Appendix.


Important: DO NOT delete your ledger directory.

## Step 1: Create a snapshot at slot 246464040
You need to stop your validator process if it is still running.

This document assumes your ledger directory is called ledger/.  If not then adjust the following commands accordingly.

Use the ledger tool to create a new snapshot at slot 246464040, replacing the two instances of <ledger path> to your actual ledger path:

```
$ solana-ledger-tool --ledger <ledger path> create-snapshot 246464040 <ledger path> --hard-fork 246464040
```

Add --snapshot-archive-path <PATH_TO_SNAPSHOTS> before “create-snapshot” if you have a separate snapshot dir, and --accounts <PATH_TO_ACCOUNTS> before -–hard-fork if you have a separate accounts dir
 
The final line of output should be “Shred version: PENDING”, and this snapshot file should now exist: 

ledger/snapshot-PENDING.tar.zst

Check your ledger/ directory to ensure that you have no snapshot newer than ledger/snapshot-PENDING.tar.zst This is very unlikely, but if found should be removed - please post on Discord if you were to find a newer snapshot! Snapshots older than ledger/snapshot-PENDING.tar.zst should not be removed.

NOTE: You may need to move the created snapshot from your ledger directory to your snapshots directory if you have a custom snapshot directory

NOTE: If you receive “Error: Slot PENDING is not available”, please see appendix


Step 2: Adjust your validator command-line arguments, temporarily for this restart to include:
(--known-validators aren’t needed if you have your own local snapshot and have set –no-genesis-fetch as your validator won’t be downloading anything, you can omit those arguments in this case)

--wait-for-supermajority PENDING \
--no-snapshot-fetch \
--no-genesis-fetch \
--expected-bank-hash PENDING\
--expected-shred-version PENDING \

(Remove the previous value of “--expected-shred-version“ if present). 

Once the cluster restarts and normal operation resumes, remember to remove --wait-for-supermajority and --expected-bank-hash before the next update or restart. They are only required for the restart. You can also go back to your old known-validators at that point.

(see next page for step 4)

Step 4: Start your validator
As it boots, it will load the snapshot for slot PENDING and wait for 80% of the stake to come online before producing/validating new blocks. 

To confirm your restarted validator is correctly waiting for 80% stake, look for this periodic log message to confirm it is waiting:
INFO  solana_core::validator] Waiting for 80% of activated stake at slot PENDING to be in gossip...

And if you have RPC enabled, ask it repeated for the current slot:
$ solana --url http://127.0.0.1:8899 slot

Any number other than PENDING means you did not complete the steps correctly.

Once started you should see log entries for “Activate stake” visible in gossip and “waiting for 80% of stake” to be visible. You can track these to see how stake progresses.


If you couldn’t produce your snapshot locally follow appendix on next page below 



Appendix: Resolution if you did not preserve your ledger or your last optimistically confirmed slot is below PENDING

NOT RECOMMENDED - this resolution should only be attempted if your ledger/ directory is unavailable or you are unable to produce a snapshot for PENDING.

ONLY IF your ledger history is corrupt or otherwise unavailable and your last confirmed slot is lower than 153139220, follow these instructions to get a new snapshot:

Your validator will need to download a new snapshot from one of the known validators. Alternative snapshot download methods are also provided further below. A snapshot will be verified as valid by the bank hash in the arguments below. 

For this you need to remove –no-snapshot-fetch if present.

Add these arguments to restart:
	--wait-for-supermajority PENDING \
--expected-shred-version PENDING \
--expected-bank-hash PENDING \


Additional snapshot sources:
If you follow this add back –no-snapshot-fetch, you can remove old snapshots, none should have a higher slot than the one you download.


