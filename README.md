# skaters-benchmarking-data

Frozen data vintages for the [skaters](https://github.com/microprediction/skaters)
benchmark. The code lives there. Only the bytes that cannot be regenerated live
here.

## Why this repo exists

Almost everything the benchmark consumes is reproducible. Shards refill from
declared studies, GIFT datasets re-download, synthetic corpora are seeded, and
model outputs resume per method and corpus. One asset is not reproducible: the
raw FRED level cache.

FRED revises series. A refetch months later returns different vintages, so a
study rerun against a fresh pull does not reproduce bit for bit. Re-acquiring
the cache also means days of throttled API calls. The archive here is the
primary copy, not a convenience copy.

## What is in the release

`fred-cache-pubdomain-YYYYMMDD.tar.zst` holds 210,574 single-series CSV files,
2.1G expanded and 759M compressed. It is attached as a release asset rather than
committed, because git is the wrong store for it and GitHub caps ordinary files
at 100MB.

The archive carries only series in the public domain. Commercial index data is
left out, and `excluded-series.txt` lists the 9,775 identifiers concerned. See
the terms section below.

## Restore

    cd ~/github/skaters
    gh release download fred-cache-20260921 \
      -R microprediction/skaters-benchmarking-data -p '*.tar.zst'
    mkdir -p data-fred-cache
    zstd -dc fred-cache-pubdomain-20260921.tar.zst | tar -x -C data-fred-cache

The benchmark reads the cache through `benchmarks/data`, a symlink to
`../data-fred-cache`. Recreate it if missing:

    ln -s ../data-fred-cache benchmarks/data

## Data provenance and terms

The MIT license in this repo covers the scripts and documentation. It does not
and cannot cover the series data, which is redistributed here unmodified from
FRED.

What ships is US government statistical output in the public domain, from
sources including BLS, BEA, Census and the Federal Reserve. Cite FRED and the
underlying agency when you use it.

Series from commercial index providers are excluded, because those providers
retain rights the benchmark cannot pass on:

| Provider | Series excluded |
| --- | --- |
| Nasdaq | 9,567 |
| ICE BofA | 192 |
| Wilshire | 16 |

Their identifiers are listed in `excluded-series.txt` and anyone with a free
FRED API key can fetch them directly. Note that a fetch today returns a current
vintage, not the one these studies ran against.

## What the exclusion costs

The benchmark's reported results are unaffected. The studies set
`STUDY_EXCLUDE_PRICE=1` and run the non-price economic universe, and the
excluded identifiers are equity and credit index series, which is precisely the
universe that flag drops.

The one stratum that does not reproduce from this archive alone is
`daily:price`, which is built from those same commercial series by the opposite
flag, `STUDY_ONLY_PRICE=1`. Refetch the excluded identifiers to rebuild it, with
the vintage caveat above.
