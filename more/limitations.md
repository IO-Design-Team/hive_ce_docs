# Limitations

- Keys have to be 32 bit unsigned integers or ASCII Strings with a max length of 255 chars.
- The supported integer values include all integers between -2^53 and 2^53, and some integers with larger magnitude
- Objects are not allowed to contain cycles. Hive will not detect them and storing will result in an infinite loop.
- Boxes are stored as files in the user's app-directory. Common illegal characters/symbols such as `/%&` should therefore be avoided.
