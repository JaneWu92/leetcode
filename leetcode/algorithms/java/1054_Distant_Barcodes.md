Solution 1:  
No pass.  
This question killed me......  
And this solution is totally wrong!!...  
It should not choose in order. It should choose by the biggest occurence.

```java
import java.util.*;

class Solution {

    private int[] result;
    private int resultIndex = -1;
    private int fIdxBegin = 0;
    private int fIdxEnd = 0;
    private int sIdxBegin = 1;
    private int sIdxEnd = 1;

    public int[] rearrangeBarcodes(int[] barcodes) {
        if (barcodes.length == 0 || barcodes.length == 1 || barcodes.length == 2) {
            return barcodes;
        }
        result = new int[barcodes.length];
        Arrays.sort(barcodes);
        for (int i = fIdxBegin; i < barcodes.length; i++) {
            if (barcodes[i] != barcodes[fIdxBegin]) {
                sIdxBegin = i;
                break;
            }
        }
        while (true) {
            for (int i = fIdxBegin; i < barcodes.length; i++) {
                if (barcodes[i] != barcodes[fIdxBegin]) {
                    fIdxEnd = (i - 1) < 0 ? 0 : (i - 1);
                    break;
                }
            }
            for (int i = sIdxBegin; i < barcodes.length; i++) {
                if (barcodes[i] != barcodes[sIdxBegin]) {
                    sIdxEnd = (i - 1) < 0 ? 0 : (i - 1);
                    break;
                }
                sIdxEnd = barcodes.length - 1;
            }
            populateResultArray(barcodes);
            if (resultIndex == barcodes.length - 1) {
                break;
            }
        }
        return result;
    }

    private void populateResultArray(int[] barcodes) {
        int firstLength = fIdxEnd - fIdxBegin + 1;
        int secondLength = sIdxEnd - sIdxBegin + 1;
        if (fIdxBegin > barcodes.length - 1) {
            for (int i = 0; i < secondLength; i++) {
                result[++resultIndex] = barcodes[sIdxBegin + i];
            }
            return;
        } else if (sIdxBegin > barcodes.length - 1) {
            for (int i = 0; i < firstLength; i++) {
                result[++resultIndex] = barcodes[fIdxBegin + i];
            }
            return;
        }
        if (firstLength <= secondLength) {
            for (int i = 0; i < firstLength; i++) {
                result[++resultIndex] = barcodes[sIdxBegin + i];
                result[++resultIndex] = barcodes[fIdxBegin + i];
            }
            fIdxBegin = sIdxBegin + firstLength;
            sIdxBegin = sIdxEnd + 1;

        } else {
            if (resultIndex == -1 || barcodes[fIdxBegin] == result[resultIndex]) {
                for (int i = 0; i < secondLength; i++) {
                    result[++resultIndex] = barcodes[fIdxBegin + i];
                    result[++resultIndex] = barcodes[sIdxBegin + i];
                }
            } else {
                for (int i = 0; i < secondLength; i++) {
                    result[++resultIndex] = barcodes[sIdxBegin + i];
                    result[++resultIndex] = barcodes[fIdxBegin + i];
                }
            }
        }
        fIdxBegin = fIdxBegin + secondLength;
        sIdxBegin = sIdxEnd + 1;
    }
}

```