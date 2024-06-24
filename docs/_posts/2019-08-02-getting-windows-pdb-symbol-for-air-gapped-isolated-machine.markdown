---
author: kernyan9
comments: true
date: 2019-08-02 03:57:41+00:00
layout: post
link: http://kernyan.com/2019/08/02/getting-windows-pdb-symbol-for-air-gapped-isolated-machine/
slug: getting-windows-pdb-symbol-for-air-gapped-isolated-machine
title: Getting window's pdb symbol for air gapped / isolated environment
wordpress_id: 1777
categories:
- C/C++
tags:
- debugging
- windows
---




Occasionally while debugging on visual studio, we encounter certain MFC functions for which no pdb symbols exist. For instance, the dll function with gibberish function names in visual studio's call stack window as seen below,





![](https://kernyan.com/wp-content/uploads/2019/08/no-symbol.png)





Those unloaded symbols are due to VS not finding the corresponding pdb for the dll used. pdb files are useful for debugging, as it shows the actual function name, and matches the function to the source file (provided that the appropriate source and header files are present in the VS solution).







However, another important need for pdb that is unrelated to the dll functions is how its absence affect visual studio's line stepping time during debugging. In those cases, having unloaded symbols severely slows down line by line debugging, which can be alleviated by closing the "Call Stack" window.







An alternative to closing the "Call Stack" window is to simply load the symbols, for MFC symbols, right clicking on the dll functions and selecting "Load Symbols" will get VS to retrieve the missing symbols from Microsoft's symbol server.





![](https://kernyan.com/wp-content/uploads/2019/08/symbol-load-information.png)





Simple.







What's tricky is figuring out how to load those symbols on an isolated environment / air gapped machine that can't connect to Microsoft's symbol server.







We could manually download the pdb files and transfer over to the isolated environment. To do that, first we need to know the pdb's name and version that we need. Obviously, the pdb's version has to match up with the dll used, which can be found by double clicking on the dll functions in the call stack window (ensure that you uncheck show disassembly in VS's debug setting beforehand, else you'll load up the disassembly instead of the dll information page below)





![](https://kernyan.com/wp-content/uploads/2019/08/version-name.png)





While we now know the right pdb version and name, we don't know where we could obtain one, since they aren't exactly listed for download by Microsoft.







For that, we could use Window's Debugger Tools [symchk.exe](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/using-symchk) to download the pdb [manually ](https://docs.microsoft.com/en-us/windows/win32/dxtecharts/debugging-with-symbols#getting-symbols-manually)(on a separate machine with internet access) by specifying a manifest file (which is a format that tells Microsoft's symbol server which pdb files we need).







The manifest file can created from the executable using symchk, but that means we have to either transfer the executable or dll from the isolated environment out, or create the manifest file in the isolated environment and then transfer the manifest file out.







If transferring files out from the isolated environment is possible, we can follow symchk's [instruction ](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/using-a-manifest-file-with-symchk)here to obtain the pdb. 







Some isolated environment have strict policies on transferring files out. In this case, our best bet is to manually construct the manifest file by using the GUID of the pdb-s we need. The GUID is a unique identifier for the name and version of the pdb.







To obtain the GUID of the pdb-s we need, we can trigger the creation of a symbol folder with the GUID as its path by clicking "Load Symbol" in the "Call Stack" of Visual Studio, and then view the GUID by clicking "Symbol Load Information". 







Copy the GUID down to paper and reconstruct the manifest file manually as below,





![](https://kernyan.com/wp-content/uploads/2019/08/manifest2.png)











Then use symchk.exe to download the pdb using the manifest file as argument, 






    
    >symchk.exe /v /im manifest.txt







By default, the downloaded pdb-s will be placed in C:\Windows\SYMBOLS\







Once you have the pdb-s, we can then transfer and load them on to the isolated environment.









