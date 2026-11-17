# ncdu — interactive disk usage analyzer

**Presented:** week week 7, Samira Aubakirova / Zhiger Bolatbek
## The problem

Our machine hit `No space left on device`. `df -h` only showed total partition usage
at 99%, while `du -sh ~/*` printed dozens of unsorted lines cluttered with permission
errors, forcing manual directory-by-directory digging.

## What we showed

```bash
ncdu ~
