# Env::Secp_Point_add

```C++
void Secp_Point_add(Secp_point& dst , const Secp_point& a , const Secp_point& b);
```
Adds two points `a` and `b` and stores result to `dst`. Addition: sets `dst = a + b`

## Parameters
* `dst ` : destination point object handle (opaque pointer)
* `a` : left hand point operand
* `b` : right hand point operand

## Return value
* none

## Notes
* `dst`, `a` and `b` don't have to be distinct

## Example 