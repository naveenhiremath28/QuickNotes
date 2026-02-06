

```
"flowID": "922e2a58-6afa-4a81-a13b-37aa76793641"
```

```
"baseUrl": "https://api-preproduction.signzy.app"
```

```
"apiKey": "44ahJYxr982JGbd8dfRG6Rz7oLEQl5Ee"
```


```json
{"flowId":"922e2a58-6afa-4a81-a13b-37aa76793641","baseUrl":"https://api-preproduction.signzy.app","apiVersion":"v2","callbackUrl":"https://subendothelial-unoccupied-cedrick.ngrok-free.dev/credential/callback/signzy","failureRedirectPath":"http://localhost:3002/dashboard/?kyc=failure","journeyLinkValidity":3600,"successRedirectPath":"http://localhost:3002/dashboard/?kyc=succes","journeySessionExpiry":600}
```

```
{"apiKey": "44ahJYxr982JGbd8dfRG6Rz7oLEQl5Ee", "clientId": "REPLACE_ME", "callbackAuthKey": "REPLACE_ME"}
```
ttps://subendothelial-unoccupied-cedrick.ngrok-free.dev


Failure:
```json
{
  "flow": {
    "flowId": "922e2a58-6afa-4a81-a13b-37aa76793641",
    "version": 1
  },
  "processingConfig": {
    "callbackUrl": "https://webhook.site/7a413947-3d0c-4854-8fb1-fcabd95320ac",
    "authKeyForCallback": "secret_key_123",
    "language": "en",
    "redirectTime": 0,
    "successRedirectUrl": "https://yourdomain.com/kyc/success",
    "failureRedirectUrl": "https://yourdomain.com/kyc/failure",
    "journeyLinkValidity": 3600,
    "journeySessionExpiry": 600
  },
  "capturedData": {
    "images": {
      "frontDoc": "https://api.persist-global.com/api/files/1586444425/download/a636881b2d584a2ca83f81e9ae0b92d9f292c611a5ec462ba84ac13dc476e772.png",
      "backDoc": "",
      "selfie": ""
    },
    "location": {
      "latitude": 12.908594099725264,
      "longitude": 77.64463259924395
    },
    "consent": false,
    "livenessConsent": false,
    "country": "IN",
    "idType": "Driving License",
    "idType2": "",
    "verificationType": "",
    "verificationType2": "",
    "autoCaptureCameraLoadDelayFront": "2.35",
    "autoCaptureCameraLoadDelayBack": "NA",
    "frontCaptureMode": "UPLOAD",
    "frontCaptureMode2": null,
    "frontCaptureMode3": null,
    "backCaptureMode": null,
    "backCaptureMode2": null,
    "backCaptureMode3": null
  },
  "matchPercentage": {
    "dateOfBirthMatch": "0%",
    "addressMatch": "NA",
    "fullNameMatch": "NA",
    "documentNumberMatch": "NA"
  },
  "documentIntelligence": {
    "completeStatus": {
      "detailsOptical": {
        "docType": "False",
        "expiry": "True",
        "imageQA": "True",
        "mrz": "True",
        "overallStatus": "False",
        "security": "True",
        "text": "True",
        "vds": "True"
      },
      "optical": "True",
      "overallStatus": "False",
      "message": "Document type not recognized",
      "severity": "High"
    },
    "extractedFieldErrors": [
      {
        "fieldName": "Surname And Given Names",
        "message": "Name not extracted",
        "errorStatus": "NOT_EXTRACTED"
      },
      {
        "fieldName": "Date of Birth",
        "message": "Date of birth not extracted",
        "errorStatus": "NOT_EXTRACTED"
      },
      {
        "fieldName": "Document Number",
        "message": "ID number not extracted",
        "errorStatus": "NOT_EXTRACTED"
      }
    ],
    "imageQuality": {
      "result": true
    },
    "idExpired": "False",
    "extractedFields": {
      "Side1": "front"
    },
    "attempts": 2
  },
  "selfieAnalysis": {
    "passiveLiveliness": {
      "status": "Not Started"
    },
    "faceMatch": {
      "status": "Not Started"
    }
  },
  "_id": "6980a81ded0278c71fd1c633",
  "clientId": "bd158519-c601-42b7-8f99-526cafaa53f1",
  "userId": "user_001",
  "journeyId": "Z0S0UYW7T2",
  "currentStatus": "TERMINATED",
  "step": "RETRIES_EXCEEDED",
  "message": "Retries Exceeded",
  "isFrontendJourneyRequired": true,
  "inQueue": false,
  "mobileStatus": false,
  "userInfo": {
    "phoneNumber": "9999999999",
    "dateOfBirth": "01/01/1995",
    "emailAddress": "user@test.com",
    "externalReferenceId": "ref_123",
    "name": {
      "firstName": "John",
      "lastName": "Doe"
    },
    "address": {
      "street": "123 Main St",
      "street2": "",
      "city": "Mumbai",
      "region": "MH",
      "postalCode": "400001",
      "country": "IN"
    },
    "id": {
      "idType": "Driving License",
      "country": "IN"
    }
  },
  "externalApiResponses": [],
  "createdAt": 1770039325370,
  "startedAt": null,
  "completedAt": 1770039360777,
  "_deleted": false,
  "deletionTimeLimit": null,
  "updatedAt": "2026-02-02T13:35:58.995Z",
  "__v": 0,
  "ipAddresses": {
    "49.207.54.70": {
      "message": "IP Check is not Enabled"
    }
  }
}
```


Success:

```json
{
  "flow": {
    "flowId": "922e2a58-6afa-4a81-a13b-37aa76793641",
    "version": 1
  },
  "processingConfig": {
    "callbackUrl": "https://webhook.site/717d6738-93dd-49ce-8eaf-7bc7a71b62bd",
    "language": "en",
    "redirectTime": 0,
    "successRedirectUrl": "http://localhost:3002/dashboard/?kyc=succes",
    "failureRedirectUrl": "http://localhost:3002/dashboard/?kyc=failure",
    "journeyLinkValidity": 3600,
    "journeySessionExpiry": 600
  },
  "capturedData": {
    "images": {
      "frontDoc": "https://api.persist-global.com/api/files/1586446144/download/551f09f9879b440a8d15cffc68501489e53fef3b0fdb4677bf791b4cedb7631a.png",
      "backDoc": "",
      "selfie": "https://api.persist-global.com/api/files/1586446146/download/6611629c4a844528aaf76331f482b72d5319164cfed24956adb57bd50a223cff.png"
    },
    "location": {
      "latitude": 12.90857862728885,
      "longitude": 77.64472599367538
    },
    "consent": true,
    "livenessConsent": false,
    "country": "IN",
    "idType": "Driving License",
    "idType2": "",
    "verificationType": "DOC",
    "verificationType2": "",
    "autoCaptureCameraLoadDelayFront": "3.89",
    "autoCaptureCameraLoadDelayBack": "NA",
    "frontCaptureMode": "UPLOAD",
    "frontCaptureMode2": null,
    "frontCaptureMode3": null,
    "backCaptureMode": null,
    "backCaptureMode2": null,
    "backCaptureMode3": null
  },
  "matchPercentage": {
    "dateOfBirthMatch": "NA",
    "addressMatch": "NA",
    "fullNameMatch": "0%",
    "documentNumberMatch": "NA"
  },
  "documentIntelligence": {
    "completeStatus": {
      "detailsOptical": {
        "docType": "True",
        "expiry": "True",
        "imageQA": "True",
        "mrz": "True",
        "overallStatus": "True",
        "security": "True",
        "text": "True",
        "vds": "True"
      },
      "optical": "True",
      "overallStatus": "True",
      "message": "success",
      "severity": "NA"
    },
    "imageQuality": {
      "result": true
    },
    "predictedIdTypeFront": {
      "DocumentName": "Driving License",
      "FDSIDList": {
        "dCountryName": "India",
        "dDescription": "dl"
      }
    },
    "idExpired": "False",
    "extractedFields": {
      "Surname And Given Names": "NAVEEN V HIREMATH",
      "names": "NAVEEN V HIREMATH",
      "Date of Birth": "28/10/2001",
      "Document Number": "KA1420210001729",
      "Date Of Expiry": "27/10/2041",
      "Address": "2ND MAIN ROAD 2ND CROSS KUVEMPU NAGARA HOSAMANE BHADRAVATI SHIMOGAK/  577301",
      "Split Address": {
        "district": [
          "Shimoga"
        ],
        "state": [
          [
            "KARNATAKA",
            "KA"
          ]
        ],
        "city": [
          "Bhadravati"
        ],
        "pincode": "577301",
        "country": [
          "IN",
          "IND",
          "INDIA"
        ],
        "addressLine": "2ND MAIN ROAD 2ND CROSS KUVEMPU NAGARA HOSAMANE K"
      },
      "Guardian's Name": "VIRUPAKSHAIAH",
      "Issue Date": "16/02/2021",
      "CDOI": "16/02/2021",
      "State": "Karnataka",
      "Side1": "front"
    },
    "attempts": 1,
    "docIntelligenceCompletedAt": "2026-02-06T07:43:02.819Z"
  },
  "selfieAnalysis": {
    "passiveLiveliness": {
      "status": "200",
      "liveness": true,
      "score": 0.99
    },
    "faceMatch": {
      "status": "200",
      "verified": true,
      "message": "Verification completed with positive result",
      "matchPercentage": "99.00%"
    },
    "additionalFaceChecks": {
      "maskDetection": {
        "status": "200",
        "result": {
          "found": false,
          "confidence": 0
        }
      },
      "faceCoverDetection": {
        "status": "200",
        "result": {
          "isFaceCovered": true,
          "faceGears": [
            "Glasses"
          ]
        }
      }
    },
    "attempts": 2,
    "selfieAnalysisCompletedAt": "2026-02-06T07:43:17.155Z"
  },
  "_id": "69859b688e847f1cd3dad3a2",
  "clientId": "bd158519-c601-42b7-8f99-526cafaa53f1",
  "userId": "965f044f-f2a7-4c36-bc27-4477b988deac",
  "journeyId": "PLHVXAW4Q1",
  "currentStatus": "COMPLETED",
  "step": "COMPLETE",
  "message": "Journey Completed",
  "isFrontendJourneyRequired": true,
  "inQueue": false,
  "mobileStatus": false,
  "userInfo": {
    "emailAddress": "naveen.hiremath@finternetlab.io",
    "externalReferenceId": "web-1770363751856",
    "name": {
      "firstName": "Charlie Brown",
      "lastName": "Brown"
    }
  },
  "externalApiResponses": [],
  "createdAt": 1770363752259,
  "startedAt": 1770363757436,
  "completedAt": 1770363799271,
  "_deleted": false,
  "deletionTimeLimit": null,
  "updatedAt": "2026-02-06T07:43:17.157Z",
  "__v": 0,
  "ipAddresses": {
    "49.207.54.70": {
      "asn": "24309",
      "isp": "ACT Fibernet",
      "countryCode": "IN",
      "region": "KARNATAKA",
      "city": "BANGALORE DIVISION",
      "organization": "Atria Convergence Technologies",
      "latitude": "13.0206267",
      "longitude": "77.6478972",
      "isCrawler": "false",
      "timezone": "IST, Asia/Kolkata",
      "mobile": "false",
      "host": "broadband.actcorp.in",
      "proxy": "true",
      "vpn": "false",
      "tor": "false",
      "recentAbuse": "false",
      "botStatus": "false",
      "fraudScore": "73",
      "address": "2JCX+752, HRBR Layout 1st Block, HRBR Layout, Kalyan Nagar, Bengaluru, Karnataka 560043, India"
    }
  }
}
```







