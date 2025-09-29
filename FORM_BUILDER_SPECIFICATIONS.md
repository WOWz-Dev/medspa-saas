# 📝 Custom Form Builder System - Detailed Specifications

## 🎯 **Overview**

The Custom Form Builder System allows Super Admins to create, manage, and deploy various types of forms throughout the MedSpa SaaS application. These forms can be linked to services, memberships, and workflows to streamline operations and ensure compliance.

---

## 🏗️ **Form Builder Architecture**

### **Form Types**
1. **Intake Forms** - Patient registration and medical history
2. **Consent Forms** - Treatment consent and liability waivers
3. **Post-Care Forms** - Post-treatment instructions and follow-up
4. **Membership Agreements** - Membership terms and conditions
5. **Service Agreements** - Service-specific terms and policies
6. **Custom Forms** - Any other business-specific forms

### **Field Types Available**
- **Text Input** - Single line text
- **Textarea** - Multi-line text
- **Email** - Email validation
- **Phone** - Phone number with formatting
- **Date** - Date picker
- **Time** - Time picker
- **Number** - Numeric input with validation
- **Checkbox** - Single or multiple selection
- **Radio Button** - Single selection from options
- **Dropdown** - Single selection from dropdown
- **File Upload** - Document/image upload
- **Signature** - Digital signature capture
- **Rating** - Star or scale rating
- **Address** - Structured address fields
- **Emergency Contact** - Contact information fields
- **Medical History** - Checkbox list of conditions
- **Allergies** - Text input for allergy information

---

## 🎨 **Form Builder Interface**

### **Drag-and-Drop Interface**
```
┌─────────────────────────────────────────────────────────┐
│ Form Builder - [Form Name]                    [Save] [Preview] │
├─────────────────────────────────────────────────────────┤
│ Field Library          │ Form Canvas                    │
│ ┌─────────────────┐    │ ┌─────────────────────────────┐ │
│ │ Text Input      │    │ │ Form Title                  │ │
│ │ Email           │    │ │ ┌─────────────────────────┐ │ │
│ │ Phone           │    │ │ │ [Text Field]            │ │ │
│ │ Date            │    │ │ └─────────────────────────┘ │ │
│ │ Checkbox        │    │ │ ┌─────────────────────────┐ │ │
│ │ Radio Button    │    │ │ │ [Email Field]           │ │ │
│ │ Dropdown        │    │ │ └─────────────────────────┘ │ │
│ │ File Upload     │    │ │ ┌─────────────────────────┐ │ │
│ │ Signature       │    │ │ │ [Checkbox Group]        │ │ │
│ │ Rating          │    │ │ └─────────────────────────┘ │ │
│ └─────────────────┘    │ └─────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### **Field Configuration Panel**
When a field is selected, show configuration options:
- **Field Label** - Display name
- **Field Name** - Internal identifier
- **Placeholder Text** - Hint text
- **Help Text** - Additional instructions
- **Validation Rules** - Required, min/max length, format
- **Default Value** - Pre-filled value
- **Options** - For radio, checkbox, dropdown fields
- **Conditional Logic** - Show/hide based on other fields
- **Styling** - Colors, size, alignment

---

## 🔗 **Form Linking System**

### **Service Linking**
- Forms can be assigned to specific services
- Forms appear during service booking process
- Forms can be required or optional
- Forms can be shown before or after service

### **Membership Linking**
- Forms can be assigned to membership plans
- Forms appear during membership signup
- Forms can be required for membership activation
- Forms can be recurring (monthly, yearly)

### **Workflow Integration**
- Forms can trigger automated workflows
- Forms can send data to external systems
- Forms can create follow-up tasks
- Forms can send notifications

---

## 📊 **Form Management Features**

### **Form Versioning**
- Track all form changes
- Maintain form history
- Rollback to previous versions
- Compare versions side-by-side

### **Form Publishing**
- Draft mode for editing
- Published mode for live use
- Scheduled publishing
- A/B testing capabilities

### **Form Analytics**
- Completion rates
- Drop-off points
- Time to complete
- User behavior tracking
- Response analysis

---

## 🎭 **User Journey Examples**

### **Patient Intake Form Journey**

#### **Step 1: Form Assignment**
- Patient books appointment for "Laser Hair Removal"
- System checks if service requires intake form
- Form appears in booking flow

#### **Step 2: Form Completion**
- Patient fills out intake form
- Required fields: Name, DOB, Medical History, Allergies
- Optional fields: Emergency Contact, Insurance Info
- Digital signature required for consent

#### **Step 3: Form Processing**
- Form data saved to patient record
- Medical staff notified of new intake
- Form data available for appointment

#### **Step 4: Follow-up**
- Post-care form sent after treatment
- Follow-up questions about results
- Satisfaction survey

### **Staff Contract Journey**

#### **Step 1: Contract Assignment**
- New staff member added to system
- Contract template assigned based on role
- Email notification sent with contract link

#### **Step 2: Contract Review**
- Staff member reviews contract
- Can request changes or clarifications
- Contract versioning tracks all changes

#### **Step 3: Digital Signature**
- Staff member signs contract digitally
- Signature timestamped and encrypted
- Contract marked as completed

#### **Step 4: Contract Storage**
- Signed contract stored securely
- Copy sent to staff member
- Contract added to employee file

---

## 🛠️ **Technical Implementation**

### **Database Schema**
```sql
-- Forms Table
CREATE TABLE forms (
  id UUID PRIMARY KEY,
  tenant_id UUID REFERENCES tenants(id),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  form_type VARCHAR(50) NOT NULL,
  status VARCHAR(20) DEFAULT 'draft',
  version INTEGER DEFAULT 1,
  created_by UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Form Fields Table
CREATE TABLE form_fields (
  id UUID PRIMARY KEY,
  form_id UUID REFERENCES forms(id),
  field_type VARCHAR(50) NOT NULL,
  field_label VARCHAR(255) NOT NULL,
  field_name VARCHAR(100) NOT NULL,
  placeholder_text TEXT,
  help_text TEXT,
  is_required BOOLEAN DEFAULT false,
  validation_rules JSONB,
  field_options JSONB,
  display_order INTEGER,
  conditional_logic JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Form Responses Table
CREATE TABLE form_responses (
  id UUID PRIMARY KEY,
  form_id UUID REFERENCES forms(id),
  patient_id UUID REFERENCES patients(id),
  staff_id UUID REFERENCES users(id),
  response_data JSONB NOT NULL,
  submitted_at TIMESTAMP DEFAULT NOW(),
  ip_address INET,
  user_agent TEXT
);

-- Form Linking Table
CREATE TABLE form_linking (
  id UUID PRIMARY KEY,
  form_id UUID REFERENCES forms(id),
  linked_type VARCHAR(50) NOT NULL, -- 'service', 'membership', 'workflow'
  linked_id UUID NOT NULL,
  is_required BOOLEAN DEFAULT false,
  display_order INTEGER,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### **API Endpoints**
```typescript
// Form Management
GET    /api/v1/forms                    // List all forms
POST   /api/v1/forms                    // Create new form
GET    /api/v1/forms/:id                // Get form details
PUT    /api/v1/forms/:id                // Update form
DELETE /api/v1/forms/:id                // Delete form
POST   /api/v1/forms/:id/publish        // Publish form
POST   /api/v1/forms/:id/unpublish      // Unpublish form

// Form Fields
GET    /api/v1/forms/:id/fields         // Get form fields
POST   /api/v1/forms/:id/fields         // Add field to form
PUT    /api/v1/forms/:id/fields/:fieldId // Update field
DELETE /api/v1/forms/:id/fields/:fieldId // Delete field

// Form Responses
GET    /api/v1/forms/:id/responses      // Get form responses
POST   /api/v1/forms/:id/responses      // Submit form response
GET    /api/v1/forms/:id/analytics      // Get form analytics

// Form Linking
GET    /api/v1/forms/:id/links          // Get form links
POST   /api/v1/forms/:id/links          // Link form to service/membership
DELETE /api/v1/forms/:id/links/:linkId  // Remove form link
```

---

## 🎨 **UI/UX Requirements**

### **Form Builder Interface**
- **Drag-and-drop** field placement
- **Real-time preview** of form
- **Responsive design** for mobile/tablet
- **Undo/redo** functionality
- **Auto-save** every 30 seconds
- **Keyboard shortcuts** for power users

### **Form Display Interface**
- **Clean, professional** design
- **Progress indicator** for long forms
- **Save and continue** functionality
- **Mobile-optimized** for patient use
- **Accessibility** compliance (WCAG 2.1)
- **Multi-language** support

### **Form Management Interface**
- **Form library** with search and filters
- **Version history** with visual diff
- **Analytics dashboard** with charts
- **Bulk operations** for form management
- **Template library** for quick creation

---

## 🔒 **Security & Compliance**

### **Data Protection**
- **Encryption** of form data at rest
- **Secure transmission** (HTTPS)
- **Access controls** based on user roles
- **Audit logging** of all form activities
- **Data retention** policies

### **Compliance Features**
- **HIPAA compliance** for medical forms
- **GDPR compliance** for data privacy
- **Digital signature** validation
- **Form versioning** for legal compliance
- **Export capabilities** for records

---

## 📈 **Analytics & Reporting**

### **Form Analytics**
- **Completion rates** by form type
- **Drop-off analysis** by field
- **Time to complete** metrics
- **User behavior** tracking
- **Response trends** over time

### **Business Intelligence**
- **Form performance** dashboards
- **Patient engagement** metrics
- **Staff efficiency** reports
- **Compliance tracking** reports
- **Custom reports** builder

---

## 🚀 **Implementation Timeline**

### **Phase 1: Core Form Builder (Weeks 1-4)**
- [ ] Basic form creation interface
- [ ] Field types implementation
- [ ] Form validation system
- [ ] Form storage and retrieval

### **Phase 2: Advanced Features (Weeks 5-8)**
- [ ] Drag-and-drop interface
- [ ] Conditional logic
- [ ] Form versioning
- [ ] Digital signatures

### **Phase 3: Integration (Weeks 9-12)**
- [ ] Service linking
- [ ] Membership linking
- [ ] Workflow integration
- [ ] Analytics dashboard

### **Phase 4: Polish & Launch (Weeks 13-16)**
- [ ] UI/UX improvements
- [ ] Performance optimization
- [ ] Security hardening
- [ ] Documentation and training

---

This form builder system will provide the flexibility and power needed to create any type of form required for MedSpa operations, while maintaining security, compliance, and ease of use.
