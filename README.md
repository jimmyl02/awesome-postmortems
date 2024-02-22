# Awesome-Postmortems

Postmortems are writeups posted by engineering organizations when something has gone wrong (incident) and explain the root cause, how they fixed it, and how they are improving systems so that the same issue will not happen again. They are both interesting to read as they provide insight into how large systems work, help engineers avoid the same mistakes, and give customers confidence the issue won't reoccur.

This repository is a simple collection of postmortems I find interesting. Also, it's sometimes hard to find these postmortems (OpenAI for example) so I hope this can also serve as an aggregator. I plan to keep this repository up to date with any new postmortems I find so feel free to make a PR and contribute any articles you find interesting!

## Postmorterms

- 02-21-2024 - [Resend Core Outage](https://resend.com/blog/incident-report-for-february-21-2024)
  - A failed database migration caused the production tables to be dropped. These were slowly recovered from backups during which the API was unavailable. The resolution was to increase database resources to increase the speed of recovery.
- 02-20-2024 - [OpenAI ChaGPT Unexpected Responses](https://status.openai.com/incidents/ssg8fh7sfyz3)
  - Due to a bad kernel deployment, specific GPU configurations would cause incorrect outputs to be returned which resulted in incorrect and incoherent tokens to be returned
- 11-15-2023 - [OpenAI API Outage](https://status.openai.com/incidents/00fpy0yxrx1q) by [OpenAI](https://openai.com/)
  - Due to an increase in traffic, routing nodes faced memory issues due to a memory expensive operations which created GC pressure. The resolution was to reuse a memory buffer as well as implement load shedding capabilities.
- 11-06-2023 - [Dicsord Authentication Outage](https://discord.com/blog/authentication-outage) by [Discord](https://discord.com)
  - Due to planned maintainence + hardware failure, a syclladb cluster responsible for authentication suffered degraded performance. Other factors like low row cache hit rates and a retry storm further exacerbated the issue. The resolution was to wait for the healthy zone to complete its rebuild to regain quorum.
- 11-03-2023 - [Cloudflare Control Plane Outage](https://blog.cloudflare.com/post-mortem-on-cloudflare-control-plane-and-analytics-outage/) by [Cloudflare](https://cloudflare.com/)
  - Cloudflare control plane was degraded as a core datacenter became unavailable due to datacenter maintainence activities. The resolution is to ensure all GA services are highly available.
- 03-21-2023 - [OpenAI API Outage](https://status.openai.com/incidents/z0tly13xsyyb) by [OpenAI](https://openai.com/)
  - Internal network connectivity issues in kubernetes cluster caused other healthy nodes to receive more traffic than they would be able to handle. Network issues also caused observability tooling to be unavailable. Resolution was to fix the networking issue, remove unhealthy nodes, and better understand behavior when hardware fails.
- 03-20-2023 - [DALL-E Web Interface Down](https://status.openai.com/incidents/4cckbrhr8hr0) by [OpenAI](https://openai.com/)
  - Hosts were unable to properly join the cluster due to an expensive GPU diagnostics command causing initial health checks to fail. This occured due to a node restart for kubernetes upgrade. Resolution was to ensure node rejoined and implement load shedding tools.
- 03-16-2023 - [ChatGPT Outage](https://status.openai.com/incidents/ds7h4z02flf5) by [OpenAI](https://openai.com/)
  - Redis had maximum connections setting misconfigured which caused failures when it reached prod. This connection limit was not hit during staging or canary. Resolution is to improve Redis monitoring to catch the problem earlier.
