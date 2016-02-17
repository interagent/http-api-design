#### Separate Concerns

Keep things simple while designing by separating the concerns between the
different parts of the request and response cycle. Keeping simple rules here
allows for greater focus on larger and harder problems.

Requests and responses will be made to address a particular resource or
collection. Use the path to indicate identity, the body to transfer the
contents and headers to communicate metadata. Query params may be used as a
means to pass header information also in edge cases, but headers are preferred
as they are more flexible and can convey more diverse information.
