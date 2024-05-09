# Information Leak

## Email

```bash
# https://github.com/Josue87/EmailFinder
emailfinder -d $DOMAIN -p http://127.0.0.1:6152 | grep -E "$DOMAIN"

# https://github.com/JoelGMSec/LeakSearch
python3 LeakSearch.py -k $DOMAIN -o leaksearch.txt
```
