Requirement already satisfied: behave in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 1)) (1.2.6)
Requirement already satisfied: python-dotenv in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 2)) (1.0.1)
Requirement already satisfied: teradatasql in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 3)) (20.0.0.24)
Requirement already satisfied: behave-html-formatter in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 4)) (0.9.10)
Requirement already satisfied: requests in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 5)) (2.32.3)
Requirement already satisfied: openpyxl in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 6)) (3.1.5)
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/pandas/
WARNING: Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/pandas/
WARNING: Retrying (Retry(total=2, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/pandas/
WARNING: Retrying (Retry(total=1, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/pandas/
WARNING: Retrying (Retry(total=0, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/pandas/
Could not fetch URL https://pypi.org/simple/pandas/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retr
ies exceeded with url: /simple/pandas/ (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))) - skipping
ERROR: Could not find a version that satisfies the requirement pandas==2.2.2 (from versions: none)
ERROR: No matching distribution found for pandas==2.2.2
Could not fetch URL https://pypi.org/simple/pip/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries
 exceeded with url: /simple/pip/ (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))) - skipping
