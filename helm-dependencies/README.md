helm dependencies update chart-a
ls chart-a/charts
helmfile template

# Helmfile template pulls deps
rm chart-a/charts/*
helmfile template
ls chart-a/charts
