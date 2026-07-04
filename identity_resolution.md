<img width="1444" height="217" alt="Screenshot 2026-07-05 at 3 41 53 AM" src="https://github.com/user-attachments/assets/71b8e31b-23b3-419b-bc04-64a225060784" />


# Step 1: Get all customers from the delta
WITH customer_delta AS (
SELECT
    internal_uuid,
    emirates_id,
    phone,
    email,
    full_name,
    dob,
    source
FROM stg_customer_profile
where execution_date>last_execution_date

UNION ALL

SELECT
    NULL,
    emirates_id,
    NULL,
    NULL,
    full_name,
    dob,
    'AECB'
FROM stg_aecb
where execution_date>last_execution_date

UNION ALL

SELECT
    NULL,
    NULL,
    phone,
    email,
    NULL,
    NULL,
    'FRAUD'
FROM stg_fraud
where execution_date>last_execution_date

UNION ALL

SELECT
    NULL,
    NULL,
    NULL,
    NULL,
    full_name,
    dob,
    'AML'
FROM stg_aml
where execution_date>last_execution_date
)


# Step 2: Resolve the delta records into customer_id
resolved_delta = []

for record in customer_delta:

    customer = None

    # Highest priority
    if record.internal_uuid is not NULL:

        customer = lookup_by_uuid(record.internal_uuid)

    # Second priority
    if customer is None and record.emirates_id is not NULL:

        customer = lookup_by_emirates_id(record.emirates_id)

    # Third priority
    if customer is None and record.phone and record.email:

        customer = lookup_by_phone_email(
                        record.phone,
                        record.email)

    # Fourth priority
    if customer is None and record.full_name and record.dob:

        customer = lookup_by_name_dob(
                        record.full_name,
                        record.dob)

    # No customer found
    if customer is None:

        customer = generate_new_customer()

    # Attach customer id

    record.customer_id = customer.customer_id

    resolved_delta.append(record)


# Step 3: Merge the resolved delta into the customer_master

MERGE INTO customer_master t

USING resolved_delta s

ON t.customer_id = s.customer_id

WHEN MATCHED THEN

UPDATE SET

    t.internal_uuid = COALESCE(t.internal_uuid, s.internal_uuid),

    t.emirates_id = COALESCE(t.emirates_id, s.emirates_id),

    t.phone = CASE

                  WHEN s.source = 'CUSTOMER'
                  THEN s.phone

                  ELSE COALESCE(t.phone, s.phone)

              END,

    t.email = CASE

                  WHEN s.source = 'CUSTOMER'
                  THEN s.email

                  ELSE COALESCE(t.email, s.email)

              END,

    t.full_name = COALESCE(t.full_name, s.full_name),
    t.dob = COALESCE(t.dob, s.dob)

WHEN NOT MATCHED THEN

INSERT (
    customer_id,
    internal_uuid,
    emirates_id,
    phone,
    email,
    full_name,
    dob
)

VALUES (
    s.customer_id,
    s.internal_uuid,
    s.emirates_id,
    s.phone,
    s.email,
    s.full_name,
    s.dob
);    
