# Healthcare API Reference Documentation

## Overview

The MedConnect Healthcare API Reference provides comprehensive technical documentation for all FHIR R4-compliant endpoints, including clinical request/response formats, healthcare authentication methods, medical error codes, and practical clinical examples.

---

## Healthcare API Design Principles

### FHIR R4 Compliance
- **Resource-Based URLs:** Endpoints represent FHIR clinical resources (Patient, Observation, Medication)
- **Standard HTTP Methods:** Semantic use of GET, POST, PUT, DELETE for clinical operations
- **FHIR Status Codes:** Healthcare-specific HTTP status codes with clinical context
- **Clinical Operations:** FHIR-defined operations for complex clinical workflows

### Healthcare Consistency Standards
- **FHIR Naming Conventions:** camelCase for FHIR properties, following HL7 standards
- **Clinical Date Formats:** FHIR dateTime format (YYYY-MM-DDTHH:MM:SS.SSSZ)
- **Medical Coding Systems:** SNOMED CT, LOINC, ICD-10, CPT for clinical data
- **FHIR Pagination:** Bundle-based pagination for clinical resource collections

### Clinical Developer Experience
- **Healthcare Patterns:** Similar clinical endpoints follow identical FHIR patterns
- **Medical Error Messages:** Clinical context with healthcare provider guidance
- **Clinical Examples:** Real-world medical scenarios with complete FHIR code samples
- **Healthcare Testing:** All clinical examples verified in HIPAA-compliant sandbox

---

## Healthcare Authentication

### SMART on FHIR Authentication

MedConnect uses SMART on FHIR authorization to authenticate clinical requests. Our implementation follows HL7 SMART App Launch Framework for healthcare applications.

#### Clinical Authorization Flows

**EHR Launch Flow** (Provider launches app from EHR)
```javascript
// Step 1: EHR initiates launch with clinical context
const launchParams = {
  iss: 'https://fhir.medconnect.com/R4',  // FHIR server issuer
  launch: 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...',  // Launch token
  client_id: 'clinical-app-12345'
};

// Step 2: Discover FHIR server capabilities
const conformance = await fetch(`${launchParams.iss}/metadata`);
const smartExtension = conformance.rest[0].security.extension.find(
  ext => ext.url === 'http://fhir-registry.smarthealthit.org/StructureDefinition/oauth-uris'
);

const authEndpoint = smartExtension.extension.find(
  ext => ext.url === 'authorize'
).valueUri;
const tokenEndpoint = smartExtension.extension.find(
  ext => ext.url === 'token'
).valueUri;

// Step 3: Request clinical authorization
const authUrl = new URL(authEndpoint);
authUrl.searchParams.set('response_type', 'code');
authUrl.searchParams.set('client_id', launchParams.client_id);
authUrl.searchParams.set('redirect_uri', 'https://your-app.com/fhir/callback');
authUrl.searchParams.set('launch', launchParams.launch);
authUrl.searchParams.set('state', generateSecureState());
authUrl.searchParams.set('aud', launchParams.iss);
authUrl.searchParams.set('scope', 'launch patient/*.read practitioner/Practitioner.read');

// Redirect to authorization server
window.location.href = authUrl.toString();
```

### Clinical Error Types

#### Healthcare Authorization Errors (`security`)
Issues with clinical access permissions and HIPAA compliance.

```json
{
  "resourceType": "OperationOutcome",
  "issue": [
    {
      "severity": "error",
      "code": "security",
      "details": {
        "coding": [
          {
            "system": "http://medconnect.com/fhir/CodeSystem/clinical-errors",
            "code": "insufficient-clinical-scope",
            "display": "Insufficient clinical authorization scope"
          }
        ]
      },
      "diagnostics": "Request requires 'patient/Observation.read' scope for accessing patient vital signs data. Current token only has 'patient/Patient.read' scope.",
      "location": ["Observation"],
      "expression": ["Observation.where(subject.reference = 'Patient/patient-789456')"]
    }
  ]
}
```

#### Clinical Data Validation Errors (`invalid`)
Issues with clinical data format and medical coding standards.

```json
{
  "resourceType": "OperationOutcome",
  "issue": [
    {
      "severity": "error",
      "code": "invalid",
      "details": {
        "coding": [
          {
            "system": "http://medconnect.com/fhir/CodeSystem/clinical-errors",
            "code": "invalid-clinical-code",
            "display": "Invalid clinical coding system"
          }
        ]
      },
      "diagnostics": "Observation code must use LOINC coding system for laboratory results. Provided code system 'http://example.com/codes' is not supported for clinical data.",
      "location": ["Observation.code.coding[0].system"],
      "expression": ["Observation.code.coding.where(system != 'http://loinc.org')"]
    }
  ]
}
```

### Clinical Error Handling Best Practices

#### Comprehensive Healthcare Error Handling
```python
import requests
from typing import Dict, Optional
import logging

class ClinicalAPIClient:
    def __init__(self, fhir_base_url: str, access_token: str):
        self.fhir_base_url = fhir_base_url
        self.access_token = access_token
        self.session = requests.Session()
        self.session.headers.update({
            'Authorization': f'Bearer {access_token}',
            'Accept': 'application/fhir+json',
            'Content-Type': 'application/fhir+json'
        })
        self.clinical_logger = logging.getLogger('clinical_api')
    
    def handle_clinical_request(self, method: str, endpoint: str, data: Optional[Dict] = None):
        """Handle clinical API request with comprehensive healthcare error handling"""
        
        try:
            url = f"{self.fhir_base_url}/{endpoint}"
            
            if method.upper() == 'GET':
                response = self.session.get(url)
            elif method.upper() == 'POST':
                response = self.session.post(url, json=data)
            elif method.upper() == 'PUT':
                response = self.session.put(url, json=data)
            elif method.upper() == 'DELETE':
                response = self.session.delete(url)
            else:
                raise ValueError(f"Unsupported HTTP method: {method}")
            
            return self._process_clinical_response(response, endpoint)
            
        except requests.exceptions.Timeout:
            return {
                'success': False,
                'error_type': 'clinical_timeout',
                'message': 'Clinical system response timeout',
                'clinical_guidance': 'Healthcare system may be experiencing high load. Retry request or contact clinical support.',
                'retry_recommended': True
            }
            
        except requests.exceptions.ConnectionError:
            return {
                'success': False,
                'error_type': 'clinical_connection_error',
                'message': 'Unable to connect to healthcare system',
                'clinical_guidance': 'Check network connectivity and healthcare system status.',
                'status_page': 'https://status.medconnect.com'
            }
            
        except Exception as e:
            self.clinical_logger.error(f"Unexpected clinical API error: {str(e)}")
            return {
                'success': False,
                'error_type': 'unexpected_clinical_error',
                'message': 'An unexpected error occurred in clinical data processing',
                'clinical_guidance': 'Contact healthcare technical support with request details.'
            }
    
    def _process_clinical_response(self, response: requests.Response, endpoint: str):
        """Process clinical API response with healthcare-specific error handling"""
        
        # Log clinical API access for HIPAA compliance
        self.clinical_logger.info(f"Clinical API access: {endpoint}, Status: {response.status_code}")
        
        if response.status_code == 200:
            return {
                'success': True,
                'data': response.json(),
                'clinical_metadata': {
                    'fhir_version': response.headers.get('X-FHIR-Version'),
                    'last_modified': response.headers.get('Last-Modified'),
                    'etag': response.headers.get('ETag')
                }
            }
            
        elif response.status_code == 401:
            return self._handle_clinical_authorization_error(response)
            
        elif response.status_code == 403:
            return self._handle_clinical_permission_error(response)
            
        elif response.status_code == 404:
            return self._handle_clinical_not_found_error(response, endpoint)
            
        elif response.status_code == 422:
            return self._handle_clinical_validation_error(response)
            
        elif response.status_code == 429:
            return self._handle_clinical_rate_limit_error(response)
            
        else:
            return self._handle_generic_clinical_error(response)
    
    def _handle_clinical_authorization_error(self, response):
        """Handle clinical authorization failures"""
        try:
            operation_outcome = response.json()
            clinical_issue = operation_outcome.get('issue', [{}])[0]
            
            return {
                'success': False,
                'error_type': 'clinical_authorization_failed',
                'message': 'Healthcare provider authorization failed',
                'fhir_details': clinical_issue.get('diagnostics'),
                'clinical_guidance': 'Re-authenticate with healthcare system using SMART on FHIR flow',
                'required_scopes': self._extract_required_scopes(clinical_issue),
                'auth_endpoint': f"{self.fhir_base_url}/auth/authorize"
            }
        except:
            return {
                'success': False,
                'error_type': 'clinical_authorization_failed',
                'message': 'Healthcare provider authorization failed',
                'clinical_guidance': 'Contact clinical system administrator for access resolution'
            }
    
    def _handle_clinical_permission_error(self, response):
        """Handle clinical permission/scope errors"""
        try:
            operation_outcome = response.json()
            clinical_issue = operation_outcome.get('issue', [{}])[0]
            
            return {
                'success': False,
                'error_type': 'insufficient_clinical_permissions',
                'message': 'Insufficient permissions for clinical data access',
                'fhir_details': clinical_issue.get('diagnostics'),
                'clinical_guidance': 'Request additional clinical scopes or verify patient consent',
                'required_scopes': self._extract_required_scopes(clinical_issue),
                'patient_consent_url': f"{self.fhir_base_url}/patient-consent"
            }
        except:
            return {
                'success': False,
                'error_type': 'insufficient_clinical_permissions',
                'message': 'Insufficient permissions for clinical data access'
            }
    
    def _handle_clinical_validation_error(self, response):
        """Handle FHIR validation errors"""
        try:
            operation_outcome = response.json()
            validation_issues = []
            
            for issue in operation_outcome.get('issue', []):
                validation_issues.append({
                    'severity': issue.get('severity'),
                    'code': issue.get('code'),
                    'details': issue.get('details', {}).get('text'),
                    'location': issue.get('location', []),
                    'clinical_guidance': self._get_validation_guidance(issue)
                })
            
            return {
                'success': False,
                'error_type': 'clinical_validation_failed',
                'message': 'Clinical data validation failed',
                'validation_issues': validation_issues,
                'clinical_guidance': 'Review FHIR resource structure and clinical coding standards',
                'fhir_specification': 'http://hl7.org/fhir/R4/'
            }
        except:
            return {
                'success': False,
                'error_type': 'clinical_validation_failed',
                'message': 'Clinical data validation failed'
            }
    
    def _get_validation_guidance(self, issue):
        """Provide clinical guidance for validation issues"""
        guidance_map = {
            'required': 'This field is required for clinical safety and workflow compliance',
            'code-invalid': 'Use standardized medical codes (SNOMED CT, LOINC, ICD-10)',
            'value': 'Clinical value must be within acceptable medical ranges',
            'reference': 'Referenced clinical resource must exist and be accessible'
        }
        
        issue_code = issue.get('code', 'unknown')
        return guidance_map.get(issue_code, 'Review FHIR specification for clinical requirements')
```

---

## Clinical Workflow Integration

### Healthcare API Operations

#### Search Patients by Clinical Criteria

```http
GET https://api.medconnect.com/fhir/R4/Patient?given=Maria&family=Rodriguez&birthdate=1975-09-22
```

**Clinical Search Parameters:**

| Parameter | Description | Clinical Use Case | Example |
|-----------|-------------|-------------------|---------|
| `given` | Given name(s) | Patient identification | `given=Maria` |
| `family` | Family name | Patient identification | `family=Rodriguez` |
| `birthdate` | Date of birth | Age verification | `birthdate=1975-09-22` |
| `identifier` | Medical record number | Direct patient lookup | `identifier=MRN-2025-789456` |
| `gender` | Administrative gender | Clinical filtering | `gender=female` |
| `active` | Active patient status | Current patients only | `active=true` |
| `general-practitioner` | Primary care provider | Provider patient lists | `general-practitioner=Practitioner/54321` |

#### Get Patient Clinical Summary

```http
GET https://api.medconnect.com/fhir/R4/Patient/patient-789456/$everything
```

**Clinical Summary Response:**
```json
{
  "resourceType": "Bundle",
  "id": "clinical-summary-patient-789456",
  "meta": {
    "lastUpdated": "2025-05-28T10:18:31Z"
  },
  "type": "searchset",
  "total": 45,
  "entry": [
    {
      "fullUrl": "https://api.medconnect.com/fhir/R4/Patient/patient-789456",
      "resource": {
        "resourceType": "Patient",
        "id": "patient-789456",
        "name": [{"family": "Rodriguez", "given": ["Maria", "Elena"]}],
        "birthDate": "1975-09-22",
        "gender": "female"
      }
    },
    {
      "fullUrl": "https://api.medconnect.com/fhir/R4/Observation/obs-bp-20250528",
      "resource": {
        "resourceType": "Observation",
        "id": "obs-bp-20250528",
        "status": "final",
        "category": [{"coding": [{"code": "vital-signs"}]}],
        "code": {"coding": [{"system": "http://loinc.org", "code": "85354-9"}]},
        "subject": {"reference": "Patient/patient-789456"},
        "effectiveDateTime": "2025-05-28T10:13:45Z",
        "component": [
          {
            "code": {"coding": [{"code": "8480-6", "display": "Systolic BP"}]},
            "valueQuantity": {"value": 142, "unit": "mmHg"}
          },
          {
            "code": {"coding": [{"code": "8462-4", "display": "Diastolic BP"}]},
            "valueQuantity": {"value": 95, "unit": "mmHg"}
          }
        ]
      }
    }
  ]
}
```

### Clinical Decision Support Integration

#### Drug Interaction Check

```http
POST https://api.medconnect.com/fhir/R4/$clinical-decision-support
```

**Request Example:**
```json
{
  "resourceType": "Parameters",
  "parameter": [
    {
      "name": "patient",
      "valueReference": {
        "reference": "Patient/patient-789456"
      }
    },
    {
      "name": "medication",
      "valueCodeableConcept": {
        "coding": [
          {
            "system": "http://www.nlm.nih.gov/research/umls/rxnorm",
            "code": "849574",
            "display": "lisinopril 10 MG Oral Tablet"
          }
        ]
      }
    },
    {
      "name": "check-type",
      "valueString": "drug-interaction"
    }
  ]
}
```

**Clinical Decision Support Response:**
```json
{
  "resourceType": "Bundle",
  "type": "collection",
  "entry": [
    {
      "resource": {
        "resourceType": "DetectedIssue",
        "status": "final",
        "category": {
          "coding": [
            {
              "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode",
              "code": "DRG",
              "display": "Drug Interaction"
            }
          ]
        },
        "severity": "moderate",
        "patient": {
          "reference": "Patient/patient-789456"
        },
        "detail": "Potential interaction between lisinopril and potassium supplements. Monitor serum potassium levels.",
        "implicated": [
          {
            "reference": "MedicationRequest/current-potassium-supplement"
          }
        ],
        "mitigation": [
          {
            "action": {
              "coding": [
                {
                  "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode",
                  "code": "LABMON",
                  "display": "Laboratory Monitoring"
                }
              ]
            },
            "date": "2025-05-28T10:18:31Z",
            "author": {
              "reference": "Device/clinical-decision-support-engine"
            }
          }
        ]
      }
    }
  ]
}
```

---

## Healthcare Testing & Validation

### Clinical Test Environment

Use clinical test credentials for healthcare development and testing:
- **Base URL:** `https://fhir-sandbox.medconnect.com/R4`
- **Test Patients:** Pre-configured patient records with clinical data
- **Synthetic Data:** All clinical data is synthetic and HIPAA-safe

### Clinical Test Data

#### Test Patient Records
```json
// Test Patient 1 - Primary Care
{
  "id": "test-patient-001",
  "identifier": [{"value": "TEST-MRN-001"}],
  "name": [{"family": "TestPatient", "given": ["Alice"]}],
  "gender": "female",
  "birthDate": "1980-01-01",
  "active": true
}

// Test Patient 2 - Chronic Conditions
{
  "id": "test-patient-002", 
  "identifier": [{"value": "TEST-MRN-002"}],
  "name": [{"family": "TestPatient", "given": ["Robert"]}],
  "gender": "male",
  "birthDate": "1965-05-15",
  "active": true
}

// Test Patient 3 - Pediatric
{
  "id": "test-patient-003",
  "identifier": [{"value": "TEST-MRN-003"}],
  "name": [{"family": "TestPatient", "given": ["Emma"]}],
  "gender": "female", 
  "birthDate": "2015-03-10",
  "active": true
}
```

#### Clinical Test Scenarios
```yaml
# Normal Vital Signs
test_vitals_normal:
  patient: test-patient-001
  blood_pressure: "118/78 mmHg"
  heart_rate: "72 bpm"
  temperature: "98.6°F"
  
# Hypertensive Crisis
test_vitals_critical:
  patient: test-patient-002
  blood_pressure: "185/110 mmHg"  
  heart_rate: "95 bpm"
  clinical_alert: true
  
# Pediatric Growth
test_pediatric_growth:
  patient: test-patient-003
  height: "110 cm"
  weight: "20 kg"
  bmi_percentile: "75th"
```

### Healthcare Integration Testing Checklist

#### Basic Clinical Integration Testing
- [ ] Create and retrieve Patient resources successfully
- [ ] Process clinical observations (vital signs, lab results)
- [ ] Handle medication prescriptions and orders
- [ ] Verify clinical decision support alerts
- [ ] Test care team and provider management

#### Healthcare Security Testing  
- [ ] SMART on FHIR authorization flow
- [ ] Clinical scope validation and enforcement
- [ ] Patient consent verification
- [ ] HIPAA audit logging functionality
- [ ] Healthcare data encryption validation

#### Clinical Workflow Testing
- [ ] Provider-initiated clinical workflows
- [ ] Patient portal integration scenarios
- [ ] Emergency access override procedures
- [ ] Clinical alert and notification delivery
- [ ] Population health data aggregation

#### Regulatory Compliance Testing
- [ ] HIPAA minimum necessary rule compliance
- [ ] Clinical data retention policy adherence
- [ ] Audit trail completeness and accuracy
- [ ] Breach detection and notification systems
- [ ] Patient rights and access request handling

---

*This healthcare API reference provides comprehensive technical documentation for integrating FHIR R4-compliant clinical systems. For additional clinical support, visit our [Healthcare Developer Portal](https://developers.medconnect.com/healthcare) or contact our clinical technical support team at clinical-api-support@medconnect.com.*