
Policies framework

WASM
AST
REGO, JSONATA


policy is like a json which will be executed on a particular event
example we will check minimum balance will minting a token which is like a rule that should satisfy to mint new token -> policy)
design(apis, schemas, how, when etc) such a way that minter/owner can create their own policies and can be used while minting, transfer
currently pre checks we have while minting is hardcoded and if anything needs to be changed then we need to modify the code, instead we can decouple like policies and triggered whenever it required


support string where user enter rego directly
keep rego which we received in request..
no need to have anchors table, instead create and dump policy in one table (policy table) have one more table where it stitches policy to primitive op


also monitor cpu usage, memory taken by policy code for execution


should we enhance premitive ops workflow or have one more workflow(if yes how premitive ops calls policy-evaluator)
do we need seperate service which takes inputs runs wasm file and gives output
possible ways to run wasm file


TODO
identities verification middleware





#383 conflicts