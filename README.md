JSON parser and generator for ISO C
===========================================

## Features

   * Written in strict ISO C
   * No dependencies, even libc is not required
   * Simple API: one function for parsing, one function for generation
   * Supports superset of JSON: allows non-quoted identifiers as object keys
   * Very fast, does one pass
   * Makes no memory allocations

## API

    int parse_json(const char *json_string, int json_string_length,
                   struct json_token *tokens_array, int size_of_tokens_array);

Tokenizes `json_string` of length `json_string_length`.
If `tokens_array` is not `NULL`, then
all `parse_json` will store tokens in the `tokens_array`. Token with type
`JSON_TYPE_EOF` marks the end of parsed tokens. JSON token is defined as:

    struct json_token {
      const char *ptr;    // Points to the beginning of the token
      int len;            // Token length
      int type;           // Type of the token, possible values below

    #define JSON_TYPE_EOF     0   // Not a real token. This is a marker
    #define JSON_TYPE_STRING  1
    #define JSON_TYPE_NUMBER  2
    #define JSON_TYPE_OBJECT  3
    #define JSON_TYPE_TRUE    4
    #define JSON_TYPE_FALSE   5
    #define JSON_TYPE_NULL    6
    #define JSON_TYPE_ARRAY   7
    };

If `tokens_array` is `NULL`, then `parse_json` just checks the validity of
the JSON string, and points where parsing stops. If `tokens_array` is not
`NULL`, it must be pre-allocated by the caller.  
Return: On success, an offset inside `json_string` is returned
where parsing has finished. On failure, a negative number is
returned, one of:

    #define JSON_STRING_INVALID           -1
    #define JSON_STRING_INCOMPLETE        -2
    #define JSON_TOKEN_ARRAY_TOO_SMALL    -3
    #define JSON_OUTPUT_BUFFER_TOO_SMALL  -4

<!-- -->

    const struct json_token *find_json_token(const struct json_token *toks,
                                             const char *path);

This is a convenience function to fetch specific values from the parsed
string. `toks` must be a valid array, successfully populated by `parse_json()`.
A `path` is a string, an accessor to the desired element of the JSON object,
as if it was written in Javascript. For example, if parsed JSON string is
"{ foo : { bar: [1, 2, 3] } }", then path "foo.bar[0]" would return a token
that points to number "1".  
Return: pointer to the found token, or NULL on failure.

## Example: parsing configuration
