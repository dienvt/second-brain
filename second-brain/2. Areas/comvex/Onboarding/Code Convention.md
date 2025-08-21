
# PR convention
[reference](https://docs.google.com/presentation/d/1gBMCYhMrAVLPwrXb9C7OBdAJbWd-uPsHKkpIP5KEWao/edit#slide=id.g24e549696e3_0_36)

# Code style 
https://github.com/comvex-jp/style-guide


Fork to new repository then work from it
branch name thì em chỉ cần để prefix `remote/REMOTE-1234-description` hoặc `remote-bff/REMOTE-1234-description`

git commit: https://www.conventionalcommits.org/en/v1.0.0/

commit prefix `feat, fix, improve, refactor, hotfix, etc…`  + `({service_name}): short description`  
`long description`
ex: 
```
fix(remote): not use the tx already commited 
- Not fire and forget goroutine in the transaction, it leads to the tx commmited or rollback before the goroutine finish
```

