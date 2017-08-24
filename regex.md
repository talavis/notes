# Regex notes

## Selection groups (find/replace in Kate)
Use (). Referenced as \1 \2 etc.
e.g.:
find: (.*) = (.*)
replace: set field \2 name = "\1"
