# Invoice Validation DSL

A Domain Specific Language (DSL) for PDF invoice validation with policy-driven processing.

## Overview

This DSL defines a comprehensive standard operating procedure (SOP) for validating PDF invoices through a policy-driven workflow. It provides structured data models, external system integrations, and a complete workflow definition for enterprise invoice processing.

**Current Version:** 3.2.0  
**Last Updated:** 2025-05-29  
**Initial Version:** 2.0.0 (2025-05-28)

## Version History

### Version 3.2.0 (2025-05-29)
- **External Configuration Management**: Business values now stored externally with 24h refresh intervals
- **Enhanced Matching Logic**: Comprehensive matching specifications for 2-way, 3-way, contract-based, and no-PO scenarios
- **YAML Anchors & References**: Improved reusability with common field definitions and acceptance criteria
- **Detailed Execution Context**: Runtime context with enhanced agent instructions and metadata
- **Advanced Policy Framework**: Hierarchical policy structure with company, vendor, accounting, and fraud detection rules
- **Conditional Workflow Logic**: Added `can_skip` conditions and conditional node execution
- **Enhanced Error Handling**: Detailed error types and recovery strategies
- **Fraud Detection**: Built-in patterns for suspicious activity monitoring
- **Agent Instructions**: AI-specific guidance for each workflow node

### Version 2.0.0 (2025-05-28)
- Initial DSL release with core workflow definition
- Basic data models for invoice processing
- External system integration specifications
- Policy-driven validation framework
- Human review and approval workflows

## Key Features

- **Policy-Driven Processing**: Flexible validation rules based on configurable policies
- **Multi-Way Document Matching**: Supports 2-way, 3-way, contract-based, and no-PO matching
- **Configurable Tolerances**: Amount and quantity tolerance policies by vendor tier
- **Automated Workflow**: From document upload through approval with human review touchpoints
- **External System Integration**: Connects to document stores, policy engines, and metadata repositories
- **External Configuration**: Business rules stored externally for easy updates without code changes
- **Fraud Detection**: Built-in patterns for suspicious activity monitoring
- **AI-Driven Execution**: Agent instructions and metadata for autonomous workflow processing

## Complete YAML Tree Structure

```yaml
# Invoice Validation DSL - Complete Tree View
# Version: 3.2.0

metadata:
  name: "Invoice Validation Execution Plan"
  version: "3.2.0"
  description: "Comprehensive DSL for invoice validation workflow"
  schema_version: "3.2"
  created_by: "Finance Automation Team"
  last_modified: "2025-05-29T10:00:00Z"
  usage_context: "AI-driven agentic workflow"

external_config:
  description: "All frequently changing business values stored externally"
  source: "config/business_rules.yaml"
  refresh_interval: "24h"
  includes:
    - amount_thresholds
    - tolerance_percentages
    - time_windows
    - approval_limits
    - vendor_classifications
    - discount_rates
    - tax_configurations

references:
  common_fields: &common_fields
    id:
      type: "string"
      required: true
      format: "UUID"
      description: "Unique identifier"
    timestamp:
      type: "timestamp"
      required: true
      format: "ISO8601"
      description: "Creation timestamp"
    version:
      type: "string"
      required: true
      description: "Schema version"
  
  base_acceptance_criteria: &base_acceptance_criteria
    - "All required fields populated"
    - "Data integrity verified"
    - "Audit trail created"
    - "Schema validation passed"
  
  standard_outputs: &standard_outputs
    success:
      status: "completed"
      next_node: "auto"
      data_persisted: true
    partial:
      status: "partial"
      requires_review: true
      next_node: "conditional"
    failure:
      status: "failed"
      retry_allowed: true
      escalation_required: true
    skip:
      status: "skipped"
      reason: "required"
      next_node: "specified"

global_context:
  description: "Runtime context flowing through entire execution"
  doctype_supported:
    - invoice
    - credit_note
    - debit_note
  vendor_categories:
    - goods_supplier
    - service_provider
    - contractor
    - consultant
  environment:
    timezone: "UTC"
    date_format: "YYYY-MM-DD"
    currency_default: "${config.base_currency}"
    locale: "en_US"

data_models:
  sop_context:
    fields:
      - context_id                    # UUID - Unique execution context
      - document_type                 # invoice | credit_note | debit_note
      - vendor_context:               # Vendor information object
          - vendor_id                 # Unique vendor identifier
          - vendor_category           # goods_supplier | service_provider | contractor
          - vendor_tier               # strategic | preferred | standard
          - preferred_vendor          # boolean flag
          - payment_terms_default     # NET30, NET60, etc.
      - org_data:                     # Organization context
          - org_id                    # Organization identifier
          - business_unit             # Business unit code
          - cost_center               # Cost center allocation
          - approval_hierarchy        # Array of approval levels
          - fiscal_period             # Current fiscal period
      - initiated_timestamp           # Workflow start time
      - sop_version                   # Version of SOP being executed

  invoice_metadata:
    fields:
      - invoice_id                    # UUID
      - invoice_number                # Vendor's invoice number
      - vendor_id                     # Link to vendor master
      - vendor_name                   # Vendor display name
      - invoice_date                  # Invoice issue date
      - due_date                      # Payment due date
      - currency                      # ISO currency code (default: AUD)
      - total_amount                  # Total invoice amount
      - tax_amount                    # Tax component
      - subtotal_amount               # Amount before tax
      - payment_terms                 # Payment terms string
      - po_number                     # Purchase order reference
      - payment_status                # pending | paid | partially_paid | overdue
      - approval_status               # pending | approved | rejected
      - extraction_confidence         # OCR/extraction confidence score
      - extracted_timestamp           # When data was extracted

  line_item:
    fields:
      - line_number                   # Sequential line number
      - description                   # Item/service description
      - quantity                      # Quantity ordered
      - unit_price                    # Price per unit
      - line_total                    # Total line amount
      - tax_code                      # Tax classification
      - gl_account                    # General ledger account
      - cost_center                   # Cost allocation
      - project_code                  # Project tracking code

  validation_result:
    fields:
      - validation_id                 # Unique validation run ID
      - rule_id                       # Applied rule identifier
      - status                        # passed | failed | warning
      - message                       # Human-readable message
      - policy_reference              # Policy that triggered rule
      - severity                      # critical | high | medium | low
      - timestamp                     # Validation timestamp

  matching_result:
    fields:
      - matching_id                   # Unique matching run ID
      - matching_type                 # two_way | three_way | contract_based
      - documents_matched             # Array of matched documents
      - matching_score                # Overall match score (0-1)
      - variances                     # Array of variance details
      - policy_applied                # Matching policy used

  document_reference:
    fields:
      - document_type                 # invoice | purchase_order | goods_receipt_note | vendor_contract
      - document_id                   # Document unique ID
      - document_number               # Business document number
      - document_date                 # Document date

  variance_detail:
    fields:
      - field_name                    # Field with variance
      - expected_value                # Expected from reference doc
      - actual_value                  # Actual from invoice
      - variance_amount               # Numeric variance
      - variance_percentage           # Percentage variance
      - within_tolerance              # Boolean tolerance check

  review_assignment:
    fields:
      - reviewer_id                   # Assigned reviewer ID
      - reviewer_role                 # Reviewer's role
      - assignment_type               # sequential | parallel
      - sequence_order                # Order in sequence
      - assigned_timestamp            # Assignment time
      - due_timestamp                 # Review deadline

external_systems:
  metastore:
    type: database
    connection_type: postgres
    connection_string: "${env.METASTORE_CONNECTION}"
    schema: invoice_metadata
    description: "Primary metadata storage"
    
  document_store:
    type: object_storage
    provider: s3
    bucket: invoice-documents
    description: "Document binary storage"
    
  knowledge_index:
    type: search_engine
    provider: elasticsearch
    endpoint: "${env.ELASTIC_ENDPOINT}"
    index_prefix: "invoice_"
    description: "Searchable knowledge base"
    
  policy_engine:
    type: rules_engine
    provider: policy_agent
    endpoint: "${env.POLICY_ENGINE_ENDPOINT}"
    namespace: tenant.policies
    description: "Centralized policy management"

policy_framework:
  company_policies:
    description: "Organization-wide business rules"
    location: "${external_config.source}/company_policies.yaml"
    categories:
      tolerance_rules:
        description: "Amount tolerance by invoice value ranges"
        config_ref: "company_policies.tolerance"
      discount_policies:
        description: "Early payment and volume discounts"
        config_ref: "company_policies.discounts"
      bulk_payment_rules:
        description: "Bulk payment eligibility"
        config_ref: "company_policies.bulk_payments"
      delegation_matrix:
        description: "Approval delegation rules"
        config_ref: "company_policies.delegation"

  vendor_policies:
    description: "Vendor-specific agreement terms"
    location: "${external_config.source}/vendor_policies.yaml"
    categories:
      payment_terms:
        description: "Vendor-specific payment terms"
        config_ref: "vendor_policies.payment_terms"
      approval_limits:
        description: "Auto-approval thresholds by vendor"
        config_ref: "vendor_policies.approval_limits"
      special_agreements:
        description: "Custom vendor agreements"
        config_ref: "vendor_policies.special_terms"

accounting_policies:
  description: "Document matching configurations"
  location: "${external_config.source}/accounting_policies.yaml"
  
  matching_types:
    two_way_matching:
      description: "Invoice to Purchase Order matching"
      required_documents:
        - invoice
        - purchase_order
      matching_characteristics:
        vendor_validation:
          field: vendor_id
          match_type: exact
          weight: 1.0
        po_reference:
          field: po_number
          match_type: exact
          weight: 0.9
        header_amount:
          field: total_amount
          match_type: tolerance
          tolerance_type: percentage
          weight: 0.8
        line_items:
          match_type: line_by_line
          strategy: best_fit
          minimum_score: 0.8
    
    three_way_matching:
      description: "Invoice to PO to Goods Receipt matching"
      required_documents:
        - invoice
        - purchase_order
        - goods_receipt_note
      matching_characteristics:
        vendor_validation:
          across_all_docs: true
        quantity_reconciliation:
          match_type: three_way_quantity
          tolerance_percent: 2
        amount_reconciliation:
          match_type: calculated
        line_items:
          match_type: three_way_line
          sequence:
            - po_to_grn
            - grn_to_invoice
    
    contract_based_matching:
      description: "Invoice to Contract matching"
      required_documents:
        - invoice
        - vendor_contract
      matching_characteristics:
        service_period_validation:
          match_type: date_range
        payment_schedule_validation:
          match_type: schedule
          validations:
            - milestone_match
            - amount_match
            - cumulative_check
    
    no_po_matching:
      description: "Invoice validation without PO"
      required_documents:
        - invoice
        - vendor_master
      matching_characteristics:
        no_po_authorization:
          required: true
        amount_limit_check:
          match_type: threshold
        expense_category_validation:
          match_type: lookup

  matching_rules_application:
    rule_selection_logic:
      - condition: "vendor.requires_three_way_match"
        apply: three_way_matching
      - condition: "vendor.type == 'service_provider' AND vendor.has_contract"
        apply: contract_based_matching
      - condition: "invoice.po_number != null"
        apply: two_way_matching
      - condition: "vendor.no_po_allowed AND invoice.amount <= vendor.no_po_limit"
        apply: no_po_matching
      - default: manual_review

  domain_understanding:
    description: "Business domain knowledge"
    categories:
      duplicate_detection:
        config_ref: "domain_rules.duplicates"
      date_validations:
        config_ref: "domain_rules.dates"
      amount_validations:
        config_ref: "domain_rules.amounts"
      vendor_validations:
        config_ref: "domain_rules.vendors"

  fraud_detection:
    description: "Fraud detection patterns"
    categories:
      pattern_detection:
        config_ref: "fraud_rules.patterns"
      threshold_monitoring:
        config_ref: "fraud_rules.thresholds"
      velocity_checks:
        config_ref: "fraud_rules.velocity"
      anomaly_detection:
        config_ref: "fraud_rules.anomalies"

execution_plan:
  id: "invoice_validation_v3"
  display_name: "Invoice Validation Workflow"
  description: "Complete invoice validation from intake to approval"
  
  nodes:
    - initialization:
        id: "initialization"
        display_name: "Initialize Execution"
        type: "start"
        metadata:
          description: "Initialize execution context"
          agent_instructions: "Entry point - establish complete context"
          critical: true
        can_skip:
          condition: false
          requires_approval: false
          reason: "Initialization is mandatory"
        node_rules:
          - "Document must exist in metastore"
          - "Vendor must be active"
          - "User has processing permissions"
          - "Execution context complete"
        acceptance_criteria:
          - "Context initialized with all fields"
          - "Document metadata retrieved"
          - "Vendor context validated"
          - "Initial policies loaded"
        execution_plan:
          - retrieve_metadata:
              id: "retrieve_metadata"
              display_name: "Retrieve Document Metadata"
              actions:
                - fetch_from_metastore:
                    operations:
                      - query_metastore
                      - validate_metadata_integrity
          - context_building:
              id: "context_building"
              display_name: "Build Execution Context"
              actions:
                - build_context:
                    operations:
                      - create_execution_context
                      - validate_vendor_status
                      - load_applicable_policies
                      - load_external_configs

    - validation_and_matching:
        id: "validation_and_matching"
        display_name: "Validation and Matching"
        type: "processing"
        depends_on: ["initialization"]
        metadata:
          description: "Apply validation rules and match documents"
          agent_instructions: "Execute all validations per policy"
        acceptance_criteria:
          - "All validation rules applied"
          - "Matching completed per policy"
          - "Variances within tolerance"
          - "Results documented"
        execution_plan:
          - policy_retrieval:
              id: "policy_retrieval"
              actions:
                - fetch_validation_policies:
                    operations:
                      - retrieve_tolerance_policies
                      - retrieve_vendor_policies
                      - retrieve_discount_policies
                      - retrieve_matching_requirements
          - field_validation:
              id: "field_validation"
              actions:
                - apply_validation_rules:
                    operations:
                      - validate_mandatory_fields
                      - validate_amount_tolerances
                      - validate_discount_eligibility
                      - validate_tax_calculations
                      - validate_gl_coding
          - document_matching:
              id: "document_matching"
              condition: "${matching_type_required != 'none'}"
              actions:
                - retrieve_matching_documents:
                    operations:
                      - identify_po_number
                      - retrieve_purchase_order
                      - retrieve_goods_receipt
                      - retrieve_vendor_contract
                - perform_matching:
                    operations:
                      - two_way_matching
                      - three_way_matching
                      - calculate_variances
                      - apply_tolerance_rules
                      - calculate_matching_score

    - review:
        id: "review"
        display_name: "Review Process"
        type: "human_task"
        depends_on: ["validation_and_matching"]
        metadata:
          description: "Human review based on policy"
          agent_instructions: "Route to appropriate reviewers"
        can_skip:
          condition: "${review_required == false}"
          requires_approval: false
          reason: "No issues requiring review"
        acceptance_criteria:
          - "Review requirements determined"
          - "Reviewers assigned per workflow"
          - "Review decisions captured"
          - "SLA compliance maintained"
        execution_plan:
          - review_policy_retrieval:
              id: "review_policy_retrieval"
          - reviewer_assignment:
              id: "reviewer_assignment"
              condition: "${review_required == true}"
          - review_execution:
              id: "review_execution"
              condition: "${review_required == true}"

    - approval:
        id: "approval"
        display_name: "Approval Workflow"
        type: "approval_workflow"
        depends_on: ["review"]
        metadata:
          description: "Multi-level approval process"
          agent_instructions: "Apply approval matrix"
        can_skip:
          condition: "${auto_approval_eligible == true}"
          requires_approval: false
          reason: "Auto-approval criteria met"
        acceptance_criteria:
          - "Approval matrix applied"
          - "Required approvals obtained"
          - "Delegation rules followed"
          - "Audit trail maintained"
        execution_plan:
          - approval_policy_retrieval:
              id: "approval_policy_retrieval"
          - approver_assignment:
              id: "approver_assignment"
              condition: "${auto_approval_eligible == false}"
          - approval_execution:
              id: "approval_execution"

# Policy Examples (from external config)
policy_examples:
  tolerance_policy:
    id: "POL-TOL-001"
    rules:
      - high_value: ">$100k non-strategic vendors"
      - medium_value: "$50k-$100k dept approval"
      - preferred_vendor: "5% tolerance"

  matching_policy:
    id: "POL-MATCH-001"
    rules:
      - goods_supplier: "3-way match >$5k"
      - service_provider: "2-way match sufficient"

  vendor_policy:
    id: "POL-VEND-001"
    vendor_id: "VND-123456"
    rules:
      - payment_terms: "2/10 NET30"
      - auto_approval: "enabled <$10k"
```

## Major Updates from Version 2.0

### Structural Improvements
- **External Configuration**: Moved business rules to external YAML files
- **YAML Anchors**: Added reusable references for common patterns
- **Conditional Logic**: Enhanced workflow with skip conditions and branching

### Enhanced Data Models
- **Global Context**: Added runtime context flowing through execution
- **Extended Metadata**: Agent instructions and critical flags for nodes
- **Output Specifications**: Detailed success/failure/partial/skip states

### Advanced Matching
- **Four Matching Types**: 2-way, 3-way, contract-based, and no-PO
- **Detailed Characteristics**: Line-by-line matching with configurable strategies
- **Tolerance Framework**: Percentage and absolute tolerance configurations

### Policy Framework
- **Hierarchical Structure**: Company → Vendor → Accounting → Domain → Fraud
- **External References**: All policies reference external configuration files
- **Dynamic Selection**: Rule-based policy application logic

### AI/Agent Support
- **Agent Instructions**: Explicit guidance for autonomous execution
- **Metadata Context**: Purpose and criticality indicators
- **Error Recovery**: Detailed error types with suggested actions

## Migration Guide from v2.0 to v3.2

1. **Extract Business Rules**: Move hardcoded values to `config/business_rules.yaml`
2. **Update Node Structure**: Add metadata, can_skip, and agent_instructions
3. **Enhance Outputs**: Convert simple outputs to structured success/failure/partial/skip
4. **Add Matching Logic**: Implement detailed matching characteristics
5. **Enable External Config**: Set up configuration refresh mechanism
