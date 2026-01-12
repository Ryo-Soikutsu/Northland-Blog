---
description: 'Solved by: Ryo Soikutsu'
---

# Q-Gates

## Challenge Description

(Unfortunately I wasn't able to get the description in time)

Category: Cryptography (Hard)

## Walkthrough

This challenge focuses on quantum cryptography, and provides a simple example using simple quantum gates. After extracting the provided 7zip file, we are given a .pkl (Python Pickle) file and a PDF document.

<figure><img src="../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

As expected, the Pickle file contained binary data, which I was able to extract using a Python script and dump to a CSV

```python
#!/usr/bin/env python3
"""
pkl_to_csv.py
Load a .pkl that contains a numpy ndarray (or similar), convert to CSV.

Usage:
    python pkl_to_csv.py input.pkl output.csv
"""

import sys
import os
import pickle
import numpy as np
import pandas as pd

def try_load(path):
    # 1) Try pickle.load
    with open(path, "rb") as f:
        try:
            obj = pickle.load(f)
            return obj, "pickle"
        except Exception:
            pass
    # 2) Try numpy.load (some people used np.save / np.savez with .pkl extension)
    try:
        obj = np.load(path, allow_pickle=True)
        return obj, "numpy.load"
    except Exception:
        pass
    raise ValueError("Could not load file with pickle or numpy.load. File may be corrupted or use an unusual format.")

def ndarray_to_dataframe(arr):
    # If it's a numpy recarray / structured array with named fields:
    if hasattr(arr, "dtype") and getattr(arr.dtype, "names", None):
        # structured dtype
        return pd.DataFrame.from_records(arr)
    # Generic numpy arrays
    if isinstance(arr, np.ndarray):
        if arr.ndim == 0:
            # scalar
            return pd.DataFrame([arr.item()])
        if arr.ndim == 1:
            # 1D: if elements are scalars -> single column
            # if elements are sequences of same length -> expand into columns
            first = arr[0]
            if np.isscalar(first):
                return pd.DataFrame(arr, columns=["value"])
            else:
                try:
                    stacked = np.vstack(arr)
                    # if stacked is 2D, create columns
                    cols = ["c{}".format(i) for i in range(stacked.shape[1])]
                    return pd.DataFrame(stacked, columns=cols)
                except Exception:
                    # fallback: convert to string per element
                    return pd.DataFrame([str(x) for x in arr], columns=["value"])
        if arr.ndim == 2:
            return pd.DataFrame(arr)
        # higher-dim: flatten final dims into columns, keep first axis as rows
        rows = arr.shape[0]
        rest = int(np.prod(arr.shape[1:]))
        reshaped = arr.reshape(rows, rest)
        cols = ["c{}".format(i) for i in range(reshaped.shape[1])]
        return pd.DataFrame(reshaped, columns=cols)
    raise TypeError("Provided object is not a numpy.ndarray")

def main(in_path, out_path, index=False):
    if not os.path.exists(in_path):
        print("Input file does not exist:", in_path, file=sys.stderr)
        sys.exit(2)

    print("Loading", in_path, " — WARNING: only load pickles you trust.")
    obj, loader = try_load(in_path)
    print("Loaded by:", loader, " type:", type(obj), " shape/dtype:",
          getattr(obj, "shape", None), getattr(getattr(obj, "dtype", None), "name", None))

    # If it's a numpy.lib.npyio.NpzFile (from np.savez), try to convert each array or pick first
    if hasattr(obj, "files"):  # np.load of .npz returns NpzFile with .files list
        keys = list(obj.files)
        if len(keys) == 1:
            arr = obj[keys[0]]
        else:
            # If multiple arrays, make dataframe by flattening each into columns (if same length)
            dfs = []
            for k in keys:
                a = obj[k]
                df = ndarray_to_dataframe(np.asarray(a))
                # prefix columns
                df.columns = [f"{k}_{c}" for c in df.columns]
                dfs.append(df)
            # align by index
            final = pd.concat(dfs, axis=1)
            final.to_csv(out_path, index=index)
            print("Wrote combined npz contents to", out_path)
            return
    else:
        arr = obj

    # If it's a pandas DataFrame already
    if isinstance(arr, pd.DataFrame):
        arr.to_csv(out_path, index=index)
        print("Wrote DataFrame to", out_path)
        return

    # If it is a pandas Series
    if isinstance(arr, pd.Series):
        arr.to_csv(out_path, index=index, header=True)
        print("Wrote Series to", out_path)
        return

    # Lists, tuples -> DataFrame
    if isinstance(arr, (list, tuple)):
        try:
            df = pd.DataFrame(arr)
            df.to_csv(out_path, index=index)
            print("Wrote list/tuple contents to", out_path)
            return
        except Exception:
            # fallback: stringify
            df = pd.DataFrame([str(x) for x in arr], columns=["value"])
            df.to_csv(out_path, index=index)
            print("Wrote list/tuple (stringified) to", out_path)
            return

    # For numpy arrays:
    if isinstance(arr, np.ndarray):
        df = ndarray_to_dataframe(arr)
        df.to_csv(out_path, index=index)
        print("Wrote ndarray to", out_path)
        return

    # Other objects: try to convert to array then dataframe
    try:
        import numpy as _np
        arr2 = _np.asarray(arr)
        df = ndarray_to_dataframe(arr2)
        df.to_csv(out_path, index=index)
        print("Wrote converted object to", out_path)
        return
    except Exception as e:
        print("Unrecognized object type — cannot convert automatically. Type:", type(obj), file=sys.stderr)
        print("Error:", e, file=sys.stderr)
        sys.exit(3)

if __name__ == "__main__":
    if len(sys.argv) < 3:
        print("Usage: python pkl_to_csv.py input.pkl output.csv", file=sys.stderr)
        sys.exit(1)
    main(sys.argv[1], sys.argv[2])


```

<figure><img src="../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

The PDF contained some details about basic quantum gates, as well as the encryption method used.

<figure><img src="../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

It took me way, way longer than I'd like to admit to figure out how to complete this challenge, due to its difficulty. I originally attempted to look up quantum calculators online, before trying to get ChatGPT to generate a Python script to decode the data.

After reading the PDF document closer, I noticed the following about the Hadamard Gate

<figure><img src="../../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>

In the matrix representation, I noticed that there was no 0 value, only +/- 1. Coincidentally, the CSV file provided also only contained +/- values of the same number. With this idea in mind, I opened CyberChef and cooked up the following recipe

<figure><img src="../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

Using Find/Replace recipes to clean the CSV data and convert it into a single column of binary values. Stripping the newline characters and decoding from binary was the last step needed for me to get the flag

<figure><img src="../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

## Conclusion

Honestly, took way longer than I would realistically admit. I had overthought the challenge and tunnel vision'd, which seems to be a recurring issue that I have to fix. Overall, still kind of cool to get to learn about quantum gates this way.
