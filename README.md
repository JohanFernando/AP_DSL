# Invoice Validation DSL

A Domain Specific Language (DSL) for PDF invoice validation with policy-driven processing.

## Overview

This DSL defines a comprehensive standard operating procedure (SOP) for validating PDF invoices through a policy-driven workflow. It provides structured data models, external system integrations, and a complete workflow definition for enterprise invoice processing.

**Version:** 2.0.0  
**Last Updated:** 2025-05-28

## Key Features

- **Policy-Driven Processing**: Flexible validation rules based on configurable policies
- **Multi-Way Document Matching**: Supports 2-way and 3-way matching with POs and GRNs
- **Configurable Tolerances**: Amount and quantity tolerance policies by vendor tier
- **Automated Workflow**: From document upload through approval with human review touchpoints
- **External System Integration**: Connects to document stores, policy engines, and metadata repositories

## Data Models

### Core Entities

- **sop_context**: Workflow context including vendor and organizational data
- **invoice_metadata**: Complete invoice header information
- **line_item**: Individual invoice line items with GL coding
- **validation_result**: Validation rule execution results
- **matching_result**: Document matching outcomes with variance tracking
- **review_assignment**: Human review task assignments

## Workflow Stages

1. **SOP Start**: Document upload and context initialization
2. **Metadata Extraction**: Extract invoice data with confidence scoring
3. **Validation & Matching**: Apply business rules and match related documents
4. **Review**: Human review based on policy requirements
5. **Approval**: Multi-level approval workflow

## External Systems

- **Metastore**: PostgreSQL database for invoice metadata
- **Document Store**: S3 object storage for PDF documents
- **Knowledge Index**: Elasticsearch for searchable invoice data
- **Policy Store**: Policy engine for dynamic rule evaluation

## Policy Categories

- Company accounting and payment policies
- Vendor-specific policies
- SOP workflow policies
- Compliance and regulatory policies

## YAML Structure

```yaml
dsl_metadata:
  name: "Invoice Validation DSL"
  version: "2.0.0"
  description: "Domain Specific Language for PDF invoice validation with policy-driven processing"
  schema_version: "2.0"

data_models:
  sop_context:
    fields:
      - context_id           # UUID
      - document_type        # invoice | credit_note
      - vendor_context:      # Vendor information
          - vendor_id
          - vendor_category
          - vendor_tier
          - preferred_vendor
          - payment_terms_default
      - org_data:           # Organization data
          - org_id
          - business_unit
          - cost_center
          - approval_hierarchy
          - fiscal_period
      - initiated_timestamp
      - sop_version

  invoice_metadata:
    fields:
      - invoice_id          # UUID
      - invoice_number
      - vendor_id
      - vendor_name
      - invoice_date
      - due_date
      - currency            # Default: AUD
      - total_amount
      - tax_amount
      - subtotal_amount
      - payment_terms
      - po_number
      - payment_status      # pending | paid | partially_paid | overdue
      - approval_status     # pending | approved | rejected
      - extraction_confidence
      - extracted_timestamp

  line_item:
    fields:
      - line_number
      - description
      - quantity
      - unit_price
      - line_total
      - tax_code
      - gl_account
      - cost_center
      - project_code

  validation_result:
    fields:
      - validation_id
      - rule_id
      - status              # passed | failed | warning
      - message
      - policy_reference
      - severity            # critical | high | medium | low
      - timestamp

  matching_result:
    fields:
      - matching_id
      - matching_type       # two_way | three_way | contract_based
      - documents_matched   # Array of document references
      - matching_score
      - variances          # Array of variance details
      - policy_applied

external_systems:
  metastore:
    type: database
    connection_type: postgres
    schema: invoice_metadata

  document_store:
    type: object_storage
    provider: s3
    bucket: invoice-documents

  knowledge_index_manager:
    type: search_engine
    provider: elasticsearch
    index: invoice_knowledge

  policy_store:
    type: policy_engine
    provider: policy_agent
    namespace: tenant.policies
    policy_categories:
      - company_accounting_payment
      - vendor_policies
      - sop_policies
      - compliance_policies
      - regulatory_policies

sop_workflow:
  id: "invoice_validation_sop"
  name: "Invoice Validation Standard Operating Procedure"
  
  sop_nodes:
    - sop_start:
        type: initialization
        sub_processes:
          - document_upload:
              actions:
                - upload_pdf_document
          - environment_understanding:
              actions:
                - extract_initial_metadata
                - build_sop_context

    - metadata_extraction:
        type: processing
        depends_on: [sop_start]

    - validation_and_matching:
        type: processing
        depends_on: [metadata_extraction]
        sub_processes:
          - policy_retrieval:
              actions:
                - fetch_validation_policies
          - field_validation:
              actions:
                - apply_validation_rules
          - document_matching:
              actions:
                - retrieve_matching_documents
                - perform_matching

    - review:
        type: human_task
        depends_on: [validation_and_matching]
        sub_processes:
          - review_policy_retrieval
          - reviewer_assignment
          - review_execution

    - approval:
        type: approval_workflow
        depends_on: [review]
        sub_processes:
          - approval_policy_retrieval
          - approver_assignment
          - approval_execution

policy_examples:
  tolerance_policy:
    rules:
      - high_value_invoices     # >$100k non-strategic vendors
      - medium_value_invoices   # $50k-$100k
      - preferred_vendor_tolerance

  matching_policy:
    rules:
      - goods_supplier_matching  # 3-way match required
      - service_provider_matching # 2-way match sufficient

  vendor_policy:
    rules:
      - payment_terms
      - invoice_tolerance
      - auto_approval_limits
```
