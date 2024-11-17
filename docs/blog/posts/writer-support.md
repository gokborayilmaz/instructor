---
authors:
  - ivanleomk
categories:
  - Writer SDK
comments: true
date: 2024-11-17
description: Announcing Writer integration with Instructor for structured outputs and enterprise AI workflows
draft: false
slug: writer-support
tags:
  - Writer
  - Enterprise AI
  - Integrations
---

# Structured Outputs with Writer now supported

We're excited to announce that `instructor` now supports [Writer](https://writer.com)'s enterprise-grade LLMs, including their latest Palmyra X 004 model. This integration enables structured outputs and enterprise AI workflows with Writer's powerful language models.

## Getting Started

Using Writer with `instructor` is straightforward

First, install `instructor` with Writer support by running `pip install instructor[writer]` in your terminal. Then, head over to [writer.com](https://writer.com) to sign up for Writer access and get an API key.

<!-- more -->

```python
import instructor
from writerai import Writer
from pydantic import BaseModel

# Initialize Writer client
client = instructor.from_writer(Writer())


class User(BaseModel):
    name: str
    age: int


# Extract structured data
user = client.chat.completions.create(
    model="palmyra-x-004",
    messages=[{"role": "user", "content": "Extract: John is 30 years old"}],
    response_model=User,
)

print(user)
#> name='John' age=30
```

We also support streaming with the Writer client using our `create_partial` method. This allows you to process responses incrementally as they arrive.

This is particularly valuable for maintaining responsive applications and delivering a smooth user experience, especially when dealing with larger responses so that users can see immediate results.

```python
import instructor
from writerai import Writer
from pydantic import BaseModel

# Initialize Writer client
client = instructor.from_writer(Writer())


text_block = """
In our recent online meeting, participants from various backgrounds joined to discuss the upcoming tech conference. The names and contact details of the participants were as follows:

- Name: John Doe, Email: johndoe@email.com, Twitter: @TechGuru44
- Name: Jane Smith, Email: janesmith@email.com, Twitter: @DigitalDiva88
- Name: Alex Johnson, Email: alexj@email.com, Twitter: @CodeMaster2023

During the meeting, we agreed on several key points. The conference will be held on March 15th, 2024, at the Grand Tech Arena located at 4521 Innovation Drive. Dr. Emily Johnson, a renowned AI researcher, will be our keynote speaker.

The budget for the event is set at $50,000, covering venue costs, speaker fees, and promotional activities. Each participant is expected to contribute an article to the conference blog by February 20th.

A follow-up meetingis scheduled for January 25th at 3 PM GMT to finalize the agenda and confirm the list of speakers.
"""


class User(BaseModel):
    name: str
    email: str
    twitter: str


class MeetingInfo(BaseModel):
    date: str
    location: str
    budget: int
    deadline: str


PartialMeetingInfo = instructor.Partial[MeetingInfo]


extraction_stream = client.chat.completions.create(
    model="palmyra-x-004",
    response_model=PartialMeetingInfo,
    messages=[
        {
            "role": "user",
            "content": f"Get the information about the meeting and the users {text_block}",
        },
    ],
    stream=True,
)  # type: ignore


for obj in extraction_stream:
    print(obj)
    #> date='March 15th, 2024' location='' budget=None deadline=None
    #> date='March 15th, 2024' location='Grand Tech Arena, 4521 Innovation' budget=None deadline=None
    #> date='March 15th, 2024' location='Grand Tech Arena, 4521 Innovation Drive' budget=50000 eadline='February 20th'
```

And that’s it! We're excited to see what you build with Instructor and Writer! If you have any questions about Writer, do check out the [Writer documentation](https://dev.writer.com/home/introduction).