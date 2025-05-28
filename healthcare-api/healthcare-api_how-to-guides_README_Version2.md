
```javascript
function buildFHIRPatientResource(patientData) {
    // Generate MRN if not provided
    const mrn = patientData.mrn || generateMedicalRecordNumber();
    
    const patient = {
        resourceType: "Patient",
        identifier: [
            {
                use: "usual",
                type: {
                    coding: [
                        {
                            system: "http://terminology.hl7.org/CodeSystem/v2-0203",
                            code: "MR",
                            display: "Medical Record Number"
                        }
                    ]
                },
                system: "http://medconnect-demo.com/patient-id",
                value: mrn,
                assigner: {
                    display: "MedConnect Healthcare System"
                }
            }
        ],
        active: true,
        name: [
            {
                use: "official",
                family: patientData.family,
                given: [patientData.given]
            }
        ],
        gender: patientData.gender,
        birthDate: patientData.birthDate
    };
    
    // Add contact information
    if (patientData.phonePrimary || patientData.email) {
        patient.telecom = [];
        
        if (patientData.phonePrimary) {
            patient.telecom.push({
                system: "phone",
                value: patientData.phonePrimary,
                use: "mobile",
                rank: 1
            });
        }
        
        if (patientData.email) {
            patient.telecom.push({
                system: "email",
                value: patientData.email,
                use: "home"
            });
        }
    }
    
    // Add address if provided
    if (patientData.addressLine1 && patientData.city && patientData.state) {
        patient.address = [
            {
                use: "home",
                type: "both",
                line: [patientData.addressLine1],
                city: patientData.city,
                state: patientData.state,
                postalCode: patientData.postalCode || "",
                country: "US",
                period: {
                    start: new Date().toISOString().split('T')[0]
                }
            }
        ];
        
        if (patientData.addressLine2) {
            patient.address[0].line.push(patientData.addressLine2);
        }
    }
    
    return patient;
}

function generateMedicalRecordNumber() {
    const year = new Date().getFullYear();
    const month = String(new Date().getMonth() + 1).padStart(2, '0');
    const random = Math.random().toString(36).substring(2, 8).toUpperCase();
    return `MRN-${year}${month}-${random}`;
}

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Healthcare API server running on port ${PORT}`);
    console.log(`FHIR Base URL: ${process.env.MEDCONNECT_FHIR_BASE_URL}`);
});
```

---

### Step 5: Testing Your Clinical Implementation (10 minutes)

#### Test with Clinical Sample Data
Use these healthcare test scenarios in your sandbox environment:

**Test Patient 1 - Adult Primary Care:**
```json
{
  "given": "Sarah",
  "family": "Johnson",
  "birthDate": "1985-03-15",
  "gender": "female",
  "phonePrimary": "+1-555-123-4567",
  "email": "sarah.johnson@email.com",
  "addressLine1": "123 Healthcare Ave",
  "city": "Medical City",
  "state": "CA",
  "postalCode": "90210"
}
```

**Test Patient 2 - Pediatric Patient:**
```json
{
  "given": "Michael",
  "family": "Chen",
  "birthDate": "2015-08-22",
  "gender": "male",
  "phonePrimary": "+1-555-987-6543",
  "email": "parent@email.com",
  "addressLine1": "456 Family Street",
  "city": "Pediatric City",
  "state": "TX",
  "postalCode": "75201",
  "emergencyFirstName": "Lisa",
  "emergencyLastName": "Chen",
  "emergencyRelationship": "N",
  "emergencyPhone": "+1-555-234-5678"
}
```

**Test Patient 3 - Elderly Patient with Complex Data:**
```json
{
  "given": "Robert",
  "middle": "William",
  "family": "Martinez",
  "birthDate": "1940-12-05",
  "gender": "male",
  "phonePrimary": "+1-555-456-7890",
  "addressLine1": "789 Senior Living Blvd",
  "addressLine2": "Unit 205",
  "city": "Elder Care City",
  "state": "FL",
  "postalCode": "33101",
  "maritalStatus": "W",
  "preferredLanguage": "es-US"
}
```

#### Clinical Verification Checklist
- [ ] Patient resource created successfully with proper FHIR structure
- [ ] Medical Record Number generated or validated correctly
- [ ] Clinical identifiers properly formatted and assigned
- [ ] Contact information stored with appropriate use codes
- [ ] Address information structured according to healthcare standards
- [ ] Emergency contact relationships coded correctly
- [ ] Clinical audit logs generated for HIPAA compliance

#### Testing Workflow
1. **Open your patient registration form** in a web browser
2. **Enter test patient data** using the clinical scenarios above
3. **Submit registration** and verify successful FHIR resource creation
4. **Check MedConnect healthcare dashboard** for patient record
5. **Verify clinical audit logs** show proper HIPAA-compliant logging
6. **Test validation errors** with invalid clinical data
7. **Verify error handling** displays appropriate healthcare guidance

---

### Troubleshooting Common Clinical Issues

#### Problem: "FHIR Validation Error" in Patient Creation
**Clinical Solution:**
```javascript
// Ensure required FHIR elements are present
function validateFHIRPatientResource(patient) {
    const requiredFields = ['resourceType', 'identifier', 'name', 'gender'];
    const missingFields = [];
    
    requiredFields.forEach(field => {
        if (!patient[field]) {
            missingFields.push(field);
        }
    });
    
    // Check identifier structure
    if (patient.identifier && patient.identifier.length > 0) {
        const mrn = patient.identifier[0];
        if (!mrn.system || !mrn.value) {
            missingFields.push('identifier.system or identifier.value');
        }
    }
    
    // Check name structure
    if (patient.name && patient.name.length > 0) {
        const name = patient.name[0];
        if (!name.family || !name.given) {
            missingFields.push('name.family or name.given');
        }
    }
    
    return {
        valid: missingFields.length === 0,
        missingFields: missingFields,
        clinicalGuidance: missingFields.length > 0 
            ? `Missing required FHIR elements: ${missingFields.join(', ')}`
            : 'FHIR Patient resource is valid'
    };
}
```

#### Problem: SMART on FHIR Authentication Issues
**Clinical Solution:**
```javascript
// Comprehensive SMART on FHIR error handling
async function handleSmartOnFHIRAuth(authError) {
    console.log('SMART on FHIR authentication error:', authError);
    
    if (authError.error === 'invalid_scope') {
        return {
            error: 'Insufficient clinical permissions',
            clinicalGuidance: 'Request patient/*.write scope for patient registration',
            requiredScopes: ['patient/Patient.write', 'patient/Patient.read'],
            authUrl: `${FHIR_BASE_URL}/auth/authorize`
        };
    }
    
    if (authError.error === 'invalid_client') {
        return {
            error: 'Invalid clinical application credentials',
            clinicalGuidance: 'Verify FHIR client ID and secret in healthcare dashboard',
            supportContact: 'clinical-support@medconnect.com'
        };
    }
    
    return {
        error: 'Healthcare authentication failed',
        clinicalGuidance: 'Contact clinical technical support for assistance'
    };
}
```

#### Problem: Clinical Data Privacy Concerns
**Clinical Solution:**
```python
# HIPAA-compliant logging that protects PHI
class HIPAACompliantLogger:
    def __init__(self):
        self.logger = logging.getLogger('clinical_audit')
        
    def log_patient_access(self, patient_id, user_id, action, ip_address):
        """Log patient access without exposing PHI"""
        
        # Hash patient ID for privacy while maintaining auditability
        patient_hash = hashlib.sha256(f"{patient_id}_salt".encode()).hexdigest()[:16]
        
        audit_entry = {
            'timestamp': datetime.utcnow().isoformat(),
            'action': action,
            'patient_hash': patient_hash,  # Not actual patient ID
            'user_id': user_id,
            'ip_address': self._mask_ip_address(ip_address),
            'session_id': self._get_session_id(),
            'compliance_verified': True
        }
        
        self.logger.info(f"CLINICAL_AUDIT: {json.dumps(audit_entry)}")
        
        # Store in HIPAA-compliant audit database
        self._store_audit_entry(audit_entry)
    
    def _mask_ip_address(self, ip_address):
        """Mask IP address for privacy while maintaining traceability"""
        if ':' in ip_address:  # IPv6
            parts = ip_address.split(':')
            return ':'.join(parts[:4] + ['****', '****', '****', '****'])
        else:  # IPv4
            parts = ip_address.split('.')
            return '.'.join(parts[:2] + ['***', '***'])
```

---

### Next Steps in Healthcare Development

#### Immediate Clinical Enhancements
1. **[Add Clinical Observations](./vital-signs-monitoring.md)** - Implement vital signs and lab results
2. **[Implement Care Team Management](./care-team-setup.md)** - Provider and care coordinator workflows
3. **[Add Clinical Decision Support](./clinical-decision-support.md)** - AI-powered clinical recommendations

#### Advanced Healthcare Features
1. **[Set Up Population Health Analytics](./population-health.md)** - Cohort analysis and quality measures
2. **[Integrate Medical Devices](./device-integration.md)** - IoT and monitoring device connectivity
3. **[Implement Telehealth Features](./telehealth-integration.md)** - Remote care and virtual consultations

#### Clinical Production Preparation
1. **[Healthcare Security Hardening](./clinical-security.md)** - Production HIPAA compliance
2. **[Clinical Performance Optimization](./clinical-performance.md)** - Scale for healthcare workloads
3. **[Healthcare Monitoring Setup](./clinical-monitoring.md)** - Patient safety and system health monitoring

---

### Healthcare Support Resources

- **Clinical Developer Documentation:** [clinical.medconnect.com](https://clinical.medconnect.com)
- **Healthcare Community Forum:** [community.medconnect.com/clinical](https://community.medconnect.com/clinical)
- **Clinical Technical Support:** clinical-support@medconnect.com
- **HIPAA Compliance Questions:** compliance@medconnect.com
- **Healthcare Status Updates:** [status.medconnect.com](https://status.medconnect.com)

*You've successfully implemented FHIR-compliant patient registration! This clinical foundation can be extended with additional healthcare features and optimizations as your clinical workflows evolve.*