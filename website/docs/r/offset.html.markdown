---
layout: "time"
page_title: "Time: time_offset"
description: |-
  Manages a offset time resource.
---

# Resource: time_offset

Manages an offset time resource, which keeps an UTC timestamp stored in the Terraform state that is offset from a locally sourced base timestamp. This prevents perpetual differences caused by using the [`timestamp()` function](https://www.terraform.io/docs/configuration/functions/timestamp.html).

-> Further manipulation of incoming or outgoing values can be accomplished with the [`formatdate()` function](https://www.terraform.io/docs/configuration/functions/formatdate.html) and the [`timeadd()` function](https://www.terraform.io/docs/configuration/functions/timeadd.html).

## Example Usage

### Basic Usage

```hcl
resource "time_offset" "example" {
  offset_days = 7
}

output "one_week_from_now" {
  value = time_offset.example.rfc3339
}
```

### Triggers Usage

```hcl
resource "time_offset" "ami_update" {
  triggers = {
    # Save the time each switch of an AMI id
    ami_id = data.aws_ami.example.id
  }

  offset_days = 7
}

resource "aws_instance" "server" {
  # Read the AMI id "through" the time_offset resource to ensure that
  # both will change together.
  ami = time_offset.ami_update.triggers.ami_id

  tags = {
    ExpirationTime = time_offset.ami_update.rfc3339
  }

  # ... (other aws_instance arguments) ...
}
```

## Argument Reference

~> **NOTE:** At least one of the `offset_` arguments must be configured.

The following arguments are optional:

* `base_rfc3339` - (Optional) Configure the base timestamp with an UTC [RFC3339 time string](https://tools.ietf.org/html/rfc3339#section-5.8) (`YYYY-MM-DDTHH:MM:SSZ`). Defaults to the current time.
* `triggers` - (Optional) Arbitrary map of values that, when changed, will trigger a new base timestamp value to be saved. See [the main provider documentation](../index.html) for more information.
* `offset_days` - (Optional) Number of days to offset the base timestamp. Conflicts with other `offset_` arguments.
* `offset_hours` - (Optional) Number of hours to offset the base timestamp. Conflicts with other `offset_` arguments.
* `offset_minutes` - (Optional) Number of minutes to offset the base timestamp. Conflicts with other `offset_` arguments.
* `offset_months` - (Optional) Number of months to offset the base timestamp. Conflicts with other `offset_` arguments.
* `offset_seconds` - (Optional) Number of seconds to offset the base timestamp. Conflicts with other `offset_` arguments.
* `offset_years` - (Optional) Number of years to offset the base timestamp. Conflicts with other `offset_` arguments.

## Attributes Reference

In addition to all arguments above, the following attributes are exported:

* `day` - Number day of offset timestamp.
* `hour` - Number hour of offset timestamp.
* `id` - UTC RFC3339 format of the base timestamp, e.g. `2020-02-12T06:36:13Z`.
* `minute` - Number minute of offset timestamp.
* `month` - Number month of offset timestamp.
* `rfc3339` - UTC RFC3339 format of the offset timestamp, e.g. `2020-02-12T06:36:13Z`.
* `second` - Number second of offset timestamp.
* `unix` - Number of seconds since epoch time, e.g. `1581489373`.
* `year` - Number year of offset timestamp.

## Import

This resource can be imported using the base UTC RFC3339 timestamp and offset years, months, days, hours, minutes, and seconds, separated by commas (`,`), e.g.

```console
$ terraform import time_offset.example 2020-02-12T06:36:13Z,0,0,7,0,0,0
```

The `triggers` argument cannot be imported.
