# Perceptual Hash

- [Perceptual Hash](#perceptual-hash)
  - [Pros \& Cons](#pros--cons)
      - [Pros](#pros)
      - [Cons](#cons)
  - [Algorithm](#algorithm)
      - [1. Grayscaling \& Resizing for DCT](#1-grayscaling--resizing-for-dct)
      - [2. Compute DCT 2 over the Matrix](#2-compute-dct-2-over-the-matrix)
      - [3. Crop \& Compute Median](#3-crop--compute-median)
      - [4. Compare each Value to the Median](#4-compare-each-value-to-the-median)
      - [5. Encoding to Hexadecimal](#5-encoding-to-hexadecimal)

The Perceptual Hash calculates is similar to the Average Hash, however there are some distinct differences. For the perceptual hash we will apply a Cosine Transform before calculating if each pixel is above or below the **median**.

## Pros & Cons

Like each other hashing algorithm, the perceptual hash algorithm has some pros and cons that are important to know and understand.

#### Pros

* Will be very resilities to various color changes that other hashes might fail to detect

#### Cons

* More expense to compute


## Algorithm

The algorithm behind average hash is quite simple and consists of the following steps:

1. Grayscale and resize the input image for DCT
2. Computing the DCT 2 for the Matrix
3. Crop the DCT matrix and compute the median
4. Calculate for each pixel if it is above or below the median
5. Encode the results into a hexadecimal string

Lets look into each step into more detail and how this crate implements them.

#### 1. Grayscaling & Resizing for DCT

The first step is to grayscale and resize the image to a given size. By default this size is 8 x 8 pixels.

> The size of the resized image is to determine the amount of detail that is captured in the hash.
> A larger size captures more detail while a smaller size will be "courser," i.e. less sensitive to
> fine changes in the image.

To get a better result we multiply the size by a given `factor`.
By default this factor is set to 4, meaning that we will rescale the image to 32 x 32 pixels before we compute the DCT.

> This factor is used to determine how much image details is discarded for hashing.
>
> For the default factor of 4, for example, 3/4 (or 75%) of high-frequency information is discarded,
> so the image hash is more robust against small, fine changes in the image.
>
> For a factor or 2, only 1/2 (or 50%) of high-frequency information is discarded, so the hash is more sensitive to fine changes.
> 1/8 for a factor of 8, will make the hash very resilient against fine changes, and so on.

As with every algorithm, the scale is configurable and you can experiment with it to find the best
possible image size and factor for your use case.

#### 2. Compute DCT 2 over the Matrix

Second step for calculating the DCT 2 for our matrix. We will do two passes here, first we will
compute the DCT alongside each column and then again over each row.

> DCT 2 is composable, meaning that we can compute it over the rows and then again over the columns
> to get the final result.

The formula that we use for DCT 2 is the one used by SciPy:

$$
y_k = 2\sum_{n=0}^{N-1} x_n\cos{\frac{\pi k (2 n + 1)}{2N}}
$$

Take the following image matrix as an example:

$$
\begin{bmatrix}
124 & 096 & 098\\
076 & 089 & 189\\
098 & 073 & 076\\
\end{bmatrix}
$$

After the first pass we will receive the following result (rounded to save some space here):

TODO: Update Matrix
$$
\begin{bmatrix}
124 & 096 & 098\\
076 & 089 & 189\\
098 & 073 & 076\\
\end{bmatrix}
$$

After the second pass we will get the following result:

TODO: Update Matrix
$$
\begin{bmatrix}
124 & 096 & 098\\
076 & 089 & 189\\
098 & 073 & 076\\
\end{bmatrix}
$$

#### 3. Crop & Compute Median

After we computed our DCT matrix, we will need to crop it and then calculate the median for the cropped matrix.

> Cropping is equivalent to removing high-frequency information in the DCT matrix (i.e. the parts cropped out).

Essentially we just crop our upscaled matrix down to the specified size. So if we assume a target size of 2 x 2 (default is 8 x 8), the matrix

$$
\begin{bmatrix}
124 & 096 & 098\\
076 & 089 & 189\\
098 & 073 & 076\\
\end{bmatrix}
$$

would look like this after cropping:

$$
\begin{bmatrix}
124 & 096\\
076 & 089\\
\end{bmatrix}
$$

After the cropping we then flatten the matrix and compute the median over it.

#### 4. Compare each Value to the Median

Now we go through our cropped matrix again and check if each pixel is above or below the calculated median.

If we assume the following cropped matrix and a given median of $96$:

$$
\begin{bmatrix}
124 & 096 & 098\\
076 & 089 & 189\\
098 & 073 & 076\\
\end{bmatrix}
$$

We would get the following resulting matrix:

$$
\begin{bmatrix}
true & false & true\\
false & false & true\\
true & false & false\\
\end{bmatrix}
$$

#### 5. Encoding to Hexadecimal

Each hasher in the crate returns an `ImageHash`-struct that holds the computed brightness matrix. The `encode`-method can then be used to encode the matrix into a hexadecimal string. You can also use the `decode`-function to decode a string back into its original brightness matrix.

The exact algorithm used to encoding the matrix is described [here](./encoding.md).