### JDE Details
```
JDE initial Env:
- PS920
- DV920
- PY920
- PD920

Developer -> Fat Client -> deployment server (DV920)
    - Dev modifies the code in fat client and pushes to deployment server
CNC -> Deployment Server -> Enterprise server (PY920)
    - CNC person take the modified code from deployment server and promotes to Enterprise server
Business -> Login to enterprise server and test the application
Once everything works fine CNC will promote the changes in Production
```

### Oracle DB
- 	Poor Performance after enabling FIPS 140-2 (Doc ID 2279002.1)
