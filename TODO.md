# TODO

Branch per chunk, merged with `--no-ff`. Tests green and ruff clean before
anything merges. Anything that was hard to get right gets a note in `docs/`.

## Setup

- [x] MNIST in `raw/`, sizes and hashes checked
- [x] `.gitignore`, `LICENSE`, `README`
- [ ] `src/cvfs/` skeleton
- [ ] `pyproject.toml`: ruff (E,F,I,B,UP,SIM,C4,RUF, line 88), pytest with
      `pythonpath = ["src"]`, `--strict-markers`, a `slow` marker
- [ ] banned-imports test: walk `src/cvfs/` with `ast`, fail on numpy & co. Then
      add `import numpy` somewhere on purpose and check it actually goes red.
- [ ] drop `requirements.txt` once pyproject covers it

## IDX loader

- [ ] `read_idx_images` / `read_idx_labels` with a `limit`, so I'm not reading
      47 MB to look at 100 digits
- [ ] header with `struct.unpack(">IIII")`, magic 2051 images / 2049 labels
- [ ] reject bad magic, short header, truncated payload, `limit <= 0`, limit past
      the count, labels outside 0-9
- [ ] errors say which file, and what was expected vs found
- [ ] tests build their own IDX bytes with `struct.pack`, including broken ones,
      so the suite runs on a clone with no data
- [ ] `docs/idx-format.md`

## Viewer

One rectangle per pixel, scale 12. Not an ASCII dump: I want to catch a
transposed image, and ASCII art hides that.

- [ ] canvas, label, prev/next, jump to index
- [ ] build the 784 rectangles once and `itemconfigure` them, or navigation crawls
- [ ] works on train and test
- [ ] flip through ~30 digits by hand
- [ ] screenshot in the README

After this I know the decoding is right, which is the reason it comes before any
math.

## Preprocessing

- [ ] `normalize_pixels`, pure, plus a test that it doesn't mutate its input
- [ ] `center_image` by centre of mass
- [ ] `one_hot_encode`
- [ ] x/255 vs x/127.5-1 at some point, same seed

## Vectors and matrices

- [ ] add, sub, mul, scalar mul, dot, sum, squared norm, argmax
- [ ] argmax ties: first index wins. Write it down before I forget.
- [ ] zeros, transpose, matvec, matmul, outer
- [ ] reject ragged matrices and mismatched shapes
- [ ] tests: hand-computed examples, empty input, non-square, transpose twice

Nested lists first because they're easy to check by eye. Flat storage later, once
there's a reference to compare against.

## Gradient checker

Before the network, not after. A wrong gradient doesn't crash, it just learns a
bit worse, and that is miserable to find at MNIST scale.

- [ ] central difference, eps 1e-5, relative error with a 1e-12 floor
- [ ] test it against a derivative I know
- [ ] test it rejects a gradient I've broken on purpose
- [ ] never check ReLU near 0, it isn't differentiable there and the check will
      fail on correct code

## Linear layer

- [ ] `Parameter` (values + grads), `zero_grad`
- [ ] forward `z = Wx + b`, weights out-by-in
- [ ] backward dW, db, dx. dx contracts over the output index and I fully expect
      to get it wrong the first time.
- [ ] gradient check all three
- [ ] `docs/backprop-linear.md`, derivation written out index by index

## Softmax and loss

- [ ] relu, sigmoid with the stable branch for x < 0
- [ ] softmax with the max subtracted so exp can't overflow
- [ ] fuse softmax and cross-entropy, gradient is just `p - onehot(y)`
- [ ] tests: sums to 1, survives logits of 1e3, shift-invariant, uniform 10-class
      gives log(10)
- [ ] `docs/softmax-cross-entropy.md`

## First model

784 -> 10. Small on purpose.

- [ ] overfit test first: 4 samples, 2 features, 2 classes, must hit ~100%. If it
      can't memorise four points something is broken, and I'd rather know in one
      second than after twenty minutes of MNIST.
- [ ] 500 train / 100 val / 5 epochs / batch 1
- [ ] loss goes down, accuracy beats 10%, nothing turns into NaN
- [ ] only then make the dataset bigger

## Training loop

- [ ] dataset with `__len__` / `__getitem__`
- [ ] batch iterator shuffling indices, not copying images
- [ ] last short batch handled properly, including the averaging
- [ ] test every sample is seen once per epoch, and that a seed reproduces the
      shuffle

## Metrics

- [ ] loss, accuracy, per-class accuracy, confusion matrix, timing
- [ ] test that evaluating doesn't touch the parameters
- [ ] curious whether 4/9 and 3/5 really do dominate the confusion matrix the way
      everyone says

## MLP

784 -> 64 -> 10.

- [ ] `Sequential`, forward then backward in reverse. That loop is basically all
      of backprop.
- [ ] Xavier and He, biases at 0
- [ ] inject a `random.Random` instead of touching the global one
- [ ] gradient check a 2-3-2 net end to end
- [ ] beat the linear baseline on the same budget
- [ ] plot first-layer activations for sigma=0.01 vs Xavier vs He

## Not falling over

- [ ] lr schedule, early stopping with patience, L2, clipping (elementwise first,
      then global norm which is the better one)
- [ ] checkpoints: JSON metadata + binary params, loadable without importing any
      training code
- [ ] store the normalization in the checkpoint. Train on x/255, serve on
      x/127.5-1, and it fails silently, just looks like the model got worse.

## Drawing app

- [ ] 280x280 canvas, clear, predict, confidence bars, inference time
- [ ] show the 28x28 the model actually sees, next to the drawing
- [ ] bounding box -> crop -> scale to 20x20 keeping aspect -> block average ->
      paste centred on centre of mass -> normalize -> predict
- [ ] average the blocks, don't subsample, or thin strokes vanish

That is what MNIST did to its own images. Skip it and the model can sit at 97% on
the test set and still be bad at reading my mouse. The 28x28 preview is how I'll
see that happening.

## Convolution

Only once the MLP is definitely correct.

- [ ] 5x5 input, 3x3 kernel, stride 1, no padding, checked by hand
- [ ] then multiple kernels, channels, padding, stride
- [ ] backward, gradient checked on something tiny
- [ ] maxpool (cache the argmax, that's where the gradient goes) and flatten
- [ ] `1x28x28 -> conv(4,3x3) -> relu -> maxpool -> flatten -> linear -> 10`
- [ ] overfit 20 samples before trying 2000
- [ ] show the 4 learned filters in the GUI

Rough count says conv is ~31k MACs forward, so not much worse than the MLP. The
pain will come from six nested Python loops, not the arithmetic. We'll see.

## Perf

Last, and only against a recorded baseline.

- [ ] cProfile before changing anything
- [ ] flat matrices, differential-tested against the nested-list version
- [ ] preallocate instead of appending
- [ ] bind attributes to locals in hot loops
- [ ] reuse gradient and activation buffers
- [ ] try `array.array('d')`. I think it loses to `list` because every read boxes
      a float, but I want the number.

## Later maybe

- augmentation with small shifts, if the drawing app disappoints
- dropout, batchnorm
- export a trained model as one self-contained .py
- same code on EMNIST letters
