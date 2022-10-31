shung

medium

# Internal `OptionMath._getPositivePlaceValues()` function do not handle values below `185`

## Summary

Internal `OptionMath._getPositivePlaceValues()` function do not handle values below `185`.

## Vulnerability Detail

`OptionMath._getPositivePlaceValues()` is a library function used by special floor and ceiling functions which are in turn used in the calculation of the strike price. However, `_getPositivePlaceValues()` function incorrectly reverts when the provided value is below `185`. This happens because a 64x64 `185` equals to a wad value with one digit, which is represented internally with one extra decimal. Since the division by 100 will return zero for a number with less than three digits, the division in the following line reverts due to division by zero.

https://github.com/sherlock-audit/2022-09-knox/blob/main/knox-contracts/contracts/libraries/OptionMath.sol#L101-L103

## Impact

This might prevent auctions from starting due to the following execution flow `VaultAdmin.initializeAuction() -> VaultInternal._setOptionParameters() -> Pricer.snapToGrid64x64() -> OptionMath.ceil64x64() -> OptionMath._getPositivePlaceValues()`.

There can also be more serious issues if this library is reused in other places.

## Code Snippet

https://github.com/sherlock-audit/2022-09-knox/blob/main/knox-contracts/contracts/libraries/OptionMath.sol#L93-L103

## Tool used

Manual Review

## Recommendation

Applying the following diff will properly handle division by zero, such that values below `185` can still be rounded up or down.

```diff
diff --git a/knox-contracts/contracts/libraries/OptionMath.sol b/knox-contracts/contracts/libraries/OptionMath.sol
index 746dd78..8ed2bbc 100644
--- a/knox-contracts/contracts/libraries/OptionMath.sol
+++ b/knox-contracts/contracts/libraries/OptionMath.sol
@@ -92,15 +92,21 @@ library OptionMath {
 
         // setup the first place value
         values[0].ruler = ruler;
-        values[0].value = (integer / values[0].ruler) % 10;
-
-        // setup the second place value
-        values[1].ruler = ruler / 10;
-        values[1].value = (integer / values[1].ruler) % 10;
-
-        // setup the third place value
-        values[2].ruler = ruler / 100;
-        values[2].value = (integer / values[2].ruler) % 10;
+        if (values[0].ruler != 0) {
+            values[0].value = (integer / values[0].ruler) % 10;
+
+            // setup the second place value
+            values[1].ruler = ruler / 10;
+            if (values[1].ruler != 0) {
+                values[1].value = (integer / values[1].ruler) % 10;
+
+                // setup the third place value
+                values[2].ruler = ruler / 100;
+                if (values[2].ruler != 0) {
+                    values[2].value = (integer / values[2].ruler) % 10;
+                }
+            }
+        }
 
         return (integer, values);
     }
```
