
npx @modelcontextprotocol/inspector

1. add necessary in project.toml (name and start command should be same)
```toml
[build-system]
requires = ["setuptools›=42", "wheel"]
build-backend = "setuptools.build_meta"

[tool.setuptools.packages.find]
where = ["src"]

[project.scripts]
agentic_terminal_start = "agentic_terminal. main:main"
```
2. poetry build -> whl and one more
3. can have one tools file and main file
4. you can test replace .whl with .zip, you can see project files
5. try to add in env remove src code, it should work -> poetry pip install . (. because `/dist`  in current dir )
6. try to use start command it should work
7. create account in Pypi have the token which is provided while creating account
8. set env
	1. $env:UV_PUBLISH_TOKEN="<paste-token>"



