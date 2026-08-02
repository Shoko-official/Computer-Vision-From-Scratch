# Computer Vision From Scratch

An MNIST digit classifier written with the Python standard library and nothing
else. No numpy, no torch, no PIL. The IDX parser, the linear algebra, the
backprop, the optimizers and the GUI all get written by hand.

The endpoint is a Tkinter window where you draw a digit with the mouse and it
tells you what it thinks you drew.

```text
IDX files → loader → linalg → linear classifier → backprop → MLP → CNN → GUI
```

## Why

sklearn does this in one line and gets 92%. That isn't the point. The point is
that by the end I want to be able to explain, without hand-waving, why softmax
subtracts its maximum, what the 2 in He initialization is compensating for, and
what a matrix product actually costs when there is no BLAS underneath.

So nothing here gets taken on faith. Gradients get checked against finite
differences, and any optimized version has to agree with the slow reference
version before it replaces it.

## Status

Nothing implemented yet, still setting up. Progress lives in [TODO.md](TODO.md).

## The rule

Runtime: standard library only. `tkinter`, `math`, `random`, `struct`,
`pathlib`, `time`, `json`, `array`, `dataclasses`.

Dev tools: `pytest`, `ruff`, `cProfile`.

Tkinter draws pixels and reads the mouse. It doesn't compute anything.

## Data

The MNIST files aren't in the repo (53 MB). Drop them in `raw/` with these
names:

```text
raw/train-images.idx3-ubyte    47040016 bytes
raw/train-labels.idx1-ubyte       60008
raw/t10k-images.idx3-ubyte      7840016
raw/t10k-labels.idx1-ubyte        10008
```

The sizes are worth a glance since the format fixes them exactly: images are
`16 + n*784` bytes, labels are `8 + n`. If yours don't match, the download got
truncated.

Some archives extract each file into a folder of the same name, and some use
dashes where I expect a dot before the extension. Flatten and rename.

If you want certainty rather than a size check:

```text
ba891046e6505d7aadcbbe25680a0738ad16aec93bde7f9b65e87a2fc25776db  train-images.idx3-ubyte
65a50cbbf4e906d70832878ad85ccda5333a97f0f4c3dd2ef09a8a9eef7101c5  train-labels.idx1-ubyte
0fa7898d509279e482958e8ce81c8e77db3f2f8254e26661ceb7762c4d494ce7  t10k-images.idx3-ubyte
ff7bcfd416de33731a308c3f266cc351222c34898ecbeaf847f06e48f7ec33f2  t10k-labels.idx1-ubyte
```

## Running

Python 3.14 with tkinter (bundled on Windows and macOS, `python3-tk` on Debian).

```bash
python -m venv .venv
.venv/bin/pip install --group dev
git config core.hooksPath hooks    # ruff and pytest before every commit
```

`Scripts/` instead of `bin/` on Windows. The dev tooling is a PEP 735 group in
`pyproject.toml`, so `--group` wants pip 25.1 or newer.

Entry points, as they land:

```bash
python -m cvfs.gui.dataset_viewer    # browse the raw dataset
python -m cvfs.cli.train             # train
python -m cvfs.cli.evaluate          # accuracy, confusion matrix
python -m cvfs.gui.application       # draw a digit
```

While working:

```bash
ruff check . --fix && ruff format . && pytest
```

The test suite doesn't need the MNIST files. It builds synthetic IDX bytes with
`struct.pack`; the few tests that read `raw/` are marked `slow`.

## Layout

```text
src/cvfs/
+-- data/            IDX parsing, dataset access, preprocessing
|-- math/            vectors, matrices, RNG, gradient checking
+-- nn/              parameters, layers, activations, losses, optimizers
|-- training/        trainer, batching, metrics
+-- serialization/   checkpoints
|-- gui/             dataset viewer, drawing app
+-- cli/             train, evaluate, inspect

docs/                derivations and notes for the parts that were hard
tests/               mirrors the package
```

`math` imports nothing from the project, `nn` never imports `training`, `gui`
and `cli` are leaves. If I end up with a circular import it means I got the
design wrong.

## Speed

Pure Python is somewhere between 100x and 1000x slower than a vectorized
backend, so the plan is built around that rather than pretending otherwise. A
784-64-10 MLP is roughly 100k multiply-accumulates per sample, which puts a
full-MNIST epoch in the tens of minutes. I'll be iterating on subsets of a few
thousand samples and only doing full runs when there's something worth
measuring.

Optimization comes last, after the tests and the gradient checks pass. Anything
I speed up gets measured against a recorded baseline, and these estimates get
replaced by real numbers once I have them.

## License

[MIT](LICENSE). Use it, fork it, sell it, whatever. The one condition is that
the copyright notice comes along, and that notice links back to
[me](https://github.com/Shoko-official) and to
[the original repo](https://github.com/Shoko-official/Computer-Vision-From-Scratch).

If it was useful to you I'd rather hear about it in an
[issue](https://github.com/Shoko-official/Computer-Vision-From-Scratch/issues)
than not hear about it at all.
