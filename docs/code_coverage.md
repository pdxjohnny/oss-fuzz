# Code Coverage

You can generate code coverage report for your project using [Clang Source-based
Code Coverage].


## Pull the latest Docker images

Docker images get regularly updated with a newer version of build tools, build
configurations, scripts, and other changes. It is recommended to use the most
recent images for a better user experience.


```bash
python infra/helper.py pull_images
```


## Build fuzz targets

Code Coverage report generation requires a special build configuration to be
used. In order to produce such build for your project, run:

```bash
python infra/helper.py build_image $project_name
python infra/helper.py build_fuzzers --sanitizer=profile $project_name
```


## Establish access to GCS

To get a good understanding of quality of the fuzz testing established for your
project, code coverage should be generated by running fuzz targets against the
corpus aggregated by OSS-Fuzz. The helper script will download the corpus
automatically using `gsutil` tool. To make it work, you need:

* Install [gsutil tool]
* Check whether you have access to the corpus for your project:

```bash
gsutil ls gs://${project_name}-corpus.clusterfuzz-external.appspot.com/
```

If you see an authorization error from the command above, run:

```bash
gcloud auth login
```

and try again. Once `gsutil` works, you can run the report generation.


## Generating code coverage reports

### Full project report

To generate code coverage report using the corpus aggregated on OSS-Fuzz, run:

```bash
python infra/helper.py profile $project_name
```

If you want to generate code coverage report using the corpus you have locally,
copy the corpus into `build/corpus/$project_name/$fuzz_target/` directories for
each fuzz target, then run:

```bash
python infra/helper.py profile --no-corpus-download $project_name
```

### Single fuzz target

You can generate a code coverage report for a particular fuzz target with
`--fuzz-target` argument:

```bash
python infra/helper.py profile --fuzz-target=<fuzz_target_name> $project_name
```

In this mode, you can specify an arbitrary corpus location for the fuzz target
via `--corpus-dir` to be used instead of the corpus downloaded from OSS-Fuzz:

```bash
python infra/helper.py profile --fuzz-target=<fuzz_target_name> --corpus-dir=<my_local_corpus_dir> $project_name
```

### Additional arguments for `llvm-cov`

You may want to use some of the options of [llvm-cov tool], for example,
`-ignore-filename-regex=` or `-tab-size=`. You can pass those to the helper
script after `--`:

```bash
python infra/helper.py profile $project_name -- -ignore-filename-regex='.*code/to/be/ignored/.*' -tab-size=2
```


[Clang Source-based Code Coverage]: https://clang.llvm.org/docs/SourceBasedCodeCoverage.html
[gsutil tool]: https://cloud.google.com/storage/docs/gsutil_install
[llvm-cov tool]: https://llvm.org/docs/CommandGuide/llvm-cov.html