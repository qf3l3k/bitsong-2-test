## BitSong 1 to 2 genesis modifier to a single validator

1. `bitsong-1-export.json` - Have been exported by using the command `bitsongd export --height 2482860 --for-zero-height > bitsong-1-export.json`
2. `bitsong-1-migrated.json` - Have been made by using the new `go-bitsong v0.42.x`, by using the command `bitsongd migrate bitsong-1-export.json --chain-id=bitsong-2-test --log_level info > bitsong-1-migrated.json`
3. `genesis.test.json` - The genesis that we are using in out test, have been made by using the script `node migrate.js`
