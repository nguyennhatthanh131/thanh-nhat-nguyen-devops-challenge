## Task 1: Too Many Things To Do

The command will do the following steps:
1. Extract order_ids from transaction-log.txt where the symbol is TSLA and side is sell
2. Send HTTP GET requests to https://example.com/api/:order_id for each of those order_ids
3. Write the output to ./output.txt

```bash
grep -E '"symbol": "TSLA".+"side": "sell"' ./transaction-log.txt | jq -r '.order_id' | xargs -I{} curl -s "https://example.com/api/{}" > ./output.txt
```

Explanation:
- `grep` searches for lines containing both "TSLA" and "sell"
- `jq -r '.order_id'` extracts just the order_id field from each matching JSON record
- `xargs -I{}` takes each order_id and substitutes it into the curl command
- `curl -s "https://example.com/api/{}"` sends a GET request to the API with each order_id
- Finally, `> ./output.txt` writes all responses to the output file
