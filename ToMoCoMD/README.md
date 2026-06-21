# ToMoCoMD files

Place ToMoCoMD runtime files here when running the workflow locally.

Expected structure:

```text
ToMoCoMD/
├── ToMoCoMD-CARDD_CLI.jar
├── headings.txt
└── chemical_datasets/
    └── *.sdf
```

Important: third-party `.jar` files should not be committed to GitHub unless their license explicitly allows redistribution. The `.gitignore` included in this repository excludes `*.jar` by default.
