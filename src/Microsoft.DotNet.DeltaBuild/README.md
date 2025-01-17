# DeltaBuild

DeltaBuild is a build tool for codebases that have many projects. By analyzing dependencies and Git history, DeltaBuild can determine a subset of projects that only need to be rebuilt for current changeset, and by doing so it allows to substantially decrease the solution build time. This is especially useful for pull-request. 
While DeltaBuild has been battle-tested, the results are best-effort not not 100% bullet-proof. 

Contributions are welcome.

## How it works

DeltaBuild relies on information provided by MSBuild and Git. 

1. Execute `dotnet restore /bl` to produce an MSBuild binary log for your feature branch and pass it to DeltaBuild.
1. DeltaBuild performs `git diff` between your feature branch and the target branch (default: `origin/main`).
1. DeltaBuild creates cross-section between information contained in the binary log and the git-diff.

To cover cases when projects are deleted, you will need to additionally provide a binary log for the target branch, i.e., you will have to execute `dotnet restore /bl` for that branch.

DeltaBuild can produce a JSON file with list of projects that need to be considered for rebuilding.


Example output for a small change.
```
Running DeltaBuild. This may take some time.
Binary log: C:\__w\1\s\msbuild.binlog
Repository path: C:\__w\1\s
Changed file: C:\__w\1\s\src\My.Component\My.Component.csproj
{
  "AffectedProjectChain": [
    "C:\\__w\\1\\s\\src\\\My.Component\\My.Component.csproj",
    "C:\\__w\\1\\s\\test\\\My.Component.Tests\\My.Component.Tests.csproj"
  ],
  "AffectedProjects": [
    "C:\\__w\\1\\s\\src\\Analyzers\\Aalyzers.csproj",
    "C:\\__w\\1\\s\\src\\\My.Component\\My.Component.csproj",
    "C:\\__w\\1\\s\\test\\\My.Component.Tests\\My.Component.Tests.csproj"
  ]
}
```

In this example, a change to `src\My.Component\My.Component.csproj` means we need to rebuild the project itself, its test project, and the analyzer that is a transitive dependency.

## How to run

To run DeltaBuild you need to provide a binary log and specify the root path of the repository:
```
dotnet-deltabuild --binlog msbuild.binlog --repository C:\_w\1\s
```

Optionally, you may provide additional information such as:
- `-d|--debug` if you'd like to see additional information.
- `-o|--output-json` to capture result in a file.
- `-bbl|--branch-binlog` the binlog for the target branch to account for changes where file or project may have been deleted.
- `-b|--branch` to specify target branch (default: `origin/main`)


## Results

Depending on size of the solution and relative change, measured speedup can vary from 25% to 85%.
