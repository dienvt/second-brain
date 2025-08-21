```JavaScript
Table "bib_predictions" {
  "id" bigint [pk, not null, increment]
  "image_id" bigint [not null]
  "bib_code" varchar(100) [default: NULL]
  "validation_bib_code" varchar(100) [default: NULL]
  "created_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "updated_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "deleted_at" datetime [default: NULL]
  "event_id" bigint [not null]
  "human_validation" varchar(20) [default: NULL]
  "is_valid" tinyint(1) [default: "0"]
  "verified" tinyint(1) [default: "0"]

Indexes {
  image_id [name: "FK_BP_Image"]
  (event_id, bib_code) [name: "idx_event_id_bib_code"]
}
}

Table "countries" {
  "id" bigint [pk, not null]
  "country_code" varchar(2) [not null, default: ""]
  "country_name" varchar(100) [not null, default: ""]
}

Table "customers" {
  "id" bigint [pk, not null, increment]
  "email" varchar(255) [not null]
  "password" varchar(255) [default: NULL]
  "google_id" varchar(255) [default: NULL]
  "facebook_id" varchar(255) [default: NULL]
  "register_type" int [not null]
  "is_verified" tinyint [not null]
  "is_active" tinyint(1) [not null]
  "first_name" varchar(255) [default: NULL]
  "last_name" varchar(255) [default: NULL]
  "gender" tinyint [default: NULL]
  "slug" varchar(255) [not null]
  "dob" datetime [default: NULL]
  "last_login" datetime [default: NULL]
  "created_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "updated_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "deleted_at" datetime [default: NULL]

Indexes {
  email [unique, name: "email_UNIQUE"]
}
}

Table "event_organizers" {
  "event_id" bigint [not null]
  "organizer_id" bigint [not null]

Indexes {
  organizer_id [name: "FK_EO_Organizer"]
  (event_id, organizer_id) [pk]
}
}

Table "events" {
  "id" bigint [pk, not null, increment]
  "name_vi" varchar(256) [not null, default: ""]
  "location" varchar(256) [not null]
  "start_time" datetime [not null]
  "end_time" datetime [not null]
  "hashtags" text
  "banner" text
  "introduce_vi" text
  "introduce_en" text
  "is_active" tinyint(1) [default: NULL]
  "status" int [default: NULL]
  "created_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "updated_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "deleted_at" datetime [default: NULL]
  "slug" text
  "expire_time" datetime [default: NULL]
  "type" int [default: "1"]
  "link" varchar(256) [default: NULL]
  "name_en" varchar(256) [not null, default: ""]
  "province" varchar(32) [default: NULL]
  "country" varchar(32) [default: NULL]
  "cover" varchar(256) [default: ""]
  "hint_en" varchar(128) [default: ""]
  "hint_vi" varchar(128) [default: ""]
}

Table "image_metadatas" {
  "id" bigint [pk, not null, increment]
  "bib_prediction_id" bigint [not null]
  "image_id" bigint [not null]
  "box_coordinates" blob
  "created_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "updated_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
}

Table "images" {
  "id" bigint [pk, not null, increment]
  "origin_url" text
  "event_id" bigint [not null]
  "for_training" tinyint(1) [default: NULL]
  "is_highlight" tinyint(1) [default: NULL]
  "source" text
  "created_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "updated_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "deleted_at" datetime [default: NULL]
  "small_url" text
  "medium_url" text
  "is_predicted" tinyint(1) [default: "0"]
  "batch" int [default: "1"]
  "watermarked_url" text
  "verified" tinyint(1) [default: "0"]

Indexes {
  event_id [name: "FK_I_Event"]
}
}

Table "import_campaigns" {
  "id" bigint [pk, not null, increment]
  "event_id" bigint [not null]
  "type" int [not null]
  "source" varchar(255) [not null]
  "destination" varchar(255) [not null]
  "total" int [not null]
  "success" int [not null]
  "fail" int [not null]
  "scale" float [not null]
  "logo_url" varchar(255) [not null]
  "logo_position_top" int [not null]
  "logo_position_left" int [not null]
  "step" varchar(255) [not null]
  "status" int [not null]
  "approve_by" varchar(255) [not null]
  "create_by" varchar(255) [not null]
  "created_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "updated_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "deleted_at" datetime [default: NULL]

Indexes {
  event_id [name: "FK_ic_event"]
}
}

Table "organizers" {
  "id" bigint [pk, not null, increment]
  "name" varchar(256) [not null]
  "logo" text
  "created_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "updated_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "deleted_at" datetime [default: NULL]
  "description" text
}

Table "participants" {
  "id" bigint [pk, not null, increment]
  "bib_code" varchar(100) [not null]
  "event_id" bigint [not null]
  "athlete_name" varchar(100) [default: NULL]
  "gender" varchar(10) [default: ""]
  "name_on_bib" varchar(100) [default: NULL]
  "created_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "updated_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "deleted_at" datetime [default: NULL]

Indexes {
  (bib_code, event_id) [unique, name: "uniqueue_key"]
  event_id [name: "FK_P_Event"]
}
}

Table "provinces" {
  "id" bigint [pk, not null]
  "name_vi" varchar(100) [not null, default: ""]
  "name_en" varchar(100) [not null, default: ""]
  "type" varchar(30) [not null]
  "country_id" bigint [not null]

Indexes {
  country_id [name: "FK_PRO_COU"]
}
}

Table "questions" {
  "id" bigint [pk, not null, increment]
  "name" varchar(64) [not null]
  "phone" varchar(16) [not null]
  "email" varchar(64) [not null]
  "content" varchar(64) [not null]
  "answered_by" varchar(64) [default: NULL]
  "created_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "updated_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "deleted_at" datetime [default: NULL]
  "status" int [default: "0"]
}

Table "schema_migrations" {
  "id" varchar(255) [pk, not null]
  "applied_at" datetime [default: NULL]
}

Table "users" {
  "id" bigint [pk, not null, increment]
  "username" varchar(255) [not null]
  "email" varchar(255) [not null]
  "password" varchar(255) [not null]
  "name" varchar(255) [not null]
  "role_id" tinyint [default: NULL]
  "is_active" tinyint(1) [not null]
  "created_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "updated_at" datetime [not null, default: `CURRENT_TIMESTAMP`]
  "deleted_at" datetime [default: NULL]

  Indexes {
    username [unique, name: "uk_username"]
  }
}

Ref:"images"."id" < "bib_predictions"."image_id"

Ref:"events"."id" < "bib_predictions"."event_id"

Ref:"events"."id" < "event_organizers"."event_id"

Ref:"organizers"."id" < "event_organizers"."organizer_id"

Ref:"events"."id" < "images"."event_id"

Ref:"events"."id" < "import_campaigns"."event_id"

Ref:"events"."id" < "participants"."event_id"

Ref:"countries"."id" < "provinces"."country_id"
```