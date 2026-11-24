# Ship Feature Workflow

This workflow turns a feature request into tickets and integrates their work before final review.

## Language

**Ready batch**:
The set of tickets whose blocking tickets have already been integrated. Tickets in the same ready batch can be worked on at the same time.
_Avoid_: Queue, phase

**Integrated ticket**:
A ticket whose work is present on the shared feature branch. Its dependents can then become ready.
_Avoid_: Pushed ticket, closed issue
