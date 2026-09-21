# Page-type and industry gap guide

Use this guide to identify **candidates**, not mandatory markup. Current official
Google documentation controls Google-specific requirements. Schema.org supports
more vocabulary than Google uses for rich results.

For every candidate:

1. Does it describe the target page’s primary content?
2. Is every value accurate and supported on the provided site?
3. Does an equivalent node already exist?
4. Can it connect to the existing graph by stable `@id`?
5. Does the relevant Google feature have content or technical policies?

If any answer is uncertain, leave the property out and report the gap.

## Universal checks

The SEO plugin commonly owns `WebSite`, `WebPage`, `Organization` or `Person`,
`ImageObject`, and `BreadcrumbList`. Improve those nodes instead of duplicating
them.

Check:

- canonical URL and stable `@id`;
- specific `WebPage` subtype where supported by the page;
- `name` and `description` consistent with visible content;
- `isPartOf` link from WebPage to WebSite;
- `about`, `mainEntity`, or `mainEntityOfPage` relationships where meaningful;
- primary image only when representative and available on the page;
- publisher/provider/author references to existing entity IDs;
- breadcrumb consistency with visible navigation.

Do not add properties merely to make the graph larger.

## Homepage

Likely existing nodes: `WebSite`, `WebPage`, and `Organization` or `Person`.

Candidate improvements:

- most specific supported organization subtype;
- canonical name, alternate name, URL, and logo;
- phone/email/contact point when visibly published;
- physical address only for a real, published location;
- verified social/profile links already linked by the site;
- `areaServed` only when the site explicitly states a service area.

Use `LocalBusiness` or a subtype only for a business with an actual local
presence supported by the site. Do not turn every service-area company into a
storefront.

## About page

Primary page type: `AboutPage`.

Candidates:

- `about` reference to the existing Organization or Person;
- founders, leadership, credentials, awards, founding date, and mission only
  when the page explicitly supports them;
- Person nodes for materially described team members, connected by stable IDs.

Do not infer employee roles or credentials from images or filenames.

## Contact page

Primary page type: `ContactPage`.

Candidates:

- `about` or `mainEntity` reference to the existing organization;
- published phone, email, and contact type;
- address and geo only for a physical location represented on the page;
- opening hours only when current hours are displayed;
- accessible contact action only when it accurately describes the visible flow.

Do not copy contact details from an external directory without permission.

## Location page

Primary entity: the specific `LocalBusiness` location or its supported subtype.

Candidates:

- unique location `@id`;
- name, canonical URL, phone, email;
- `PostalAddress`;
- `GeoCoordinates` only from on-site data or explicitly approved research;
- opening hours;
- parent organization reference;
- contained department or services only when the page describes them.

Represent each branch separately. Do not combine different addresses or hours.

## Service page

Primary entity: `Service`.

Candidates:

- name and concise description from the page;
- canonical URL;
- `provider` reference to the existing Organization/Person;
- `areaServed`, `audience`, `serviceType`, or `offers` only when explicit;
- `mainEntityOfPage` reference to the WebPage;
- visible FAQ represented in the SEO plugin’s existing graph shape where
  appropriate.

Do not create prices, offers, guarantees, or geographic coverage from implication.
`Service` validity does not itself promise a Google rich result.

## FAQ page or FAQ section

Primary/supporting entity: `FAQPage` with `Question` and `acceptedAnswer`.

Requirements:

- each question and answer is visible and accessible on the target page;
- the site supplies the answer (not an open user forum);
- answer text accurately reflects the visible answer;
- only questions on the target page are included.

Inspect the plugin output before choosing a shape. Yoast FAQ blocks may add
`FAQPage` to the WebPage node’s `@type` and emit linked `Question` nodes rather
than a separate FAQPage node. Reuse that graph; do not duplicate it.

Google limits FAQ rich-result visibility and may change eligibility. Implement
for accurate structured meaning, not as a display guarantee.

## Article, news, or blog post

Primary entity: `Article`, `BlogPosting`, or a more specific supported subtype.

Candidates:

- headline;
- canonical URL / `mainEntityOfPage`;
- published and modified dates from WordPress;
- author Person or Organization with a stable ID;
- publisher reference;
- representative image with dimensions when available;
- article section and keywords only when sourced;
- `about` references for clearly central entities.

Do not replace accurate plugin-generated Article schema. Fill only supported
gaps. Never fabricate an individual author when the site publishes as an
organization.

## Profile or staff page

Primary page type: `ProfilePage`; primary entity: `Person` when the page is
substantially about that person.

Candidates:

- name, role, employer affiliation;
- image;
- credentials, education, specialties, and profile links only when published;
- `mainEntity` / `mainEntityOfPage` connection.

Avoid sensitive personal data and inferred identity attributes.

## Product page

Primary entity: `Product`.

Candidates:

- name, image, description, SKU/brand when visible;
- `Offer` with current price, currency, availability, condition, and URL;
- genuine aggregate ratings/reviews represented on the page;
- shipping/return data only when current and supported.

Never invent offer data or ratings. For Google merchant listings and product
snippets, consult current Google Product documentation for required properties.
Prefer the commerce plugin’s native Product graph when it already owns the data.

## Event page

Primary entity: the most specific supported `Event` subtype.

Candidates:

- name, start/end time including timezone;
- status and attendance mode;
- physical or virtual location;
- image, description, organizer, performer;
- ticket offer details when visible.

Do not mark a general recurring-program page as one event unless dates and event
identity are clear.

## Landing page

Primary type depends on the actual offer: commonly `WebPage`, `Service`,
`Product`, `Event`, or `Course`.

- Describe the visible conversion offer.
- Use FAQ only for visible questions.
- Do not mark the form itself as a product or action unless the semantics are
  accurate.
- Respect `noindex`; structured data does not override robots directives.

## Industry refinement

Choose an industry subtype only when the site identifies the entity accordingly.
Subtypes improve specificity but increase the cost of a wrong claim.

### Healthcare

Possible types include `MedicalOrganization`, `MedicalClinic`, `Physician`, and
medical specialties. Require explicit on-site evidence for entity type,
credentials, specialties, and locations. Do not derive medical claims or
conditions from generic marketing language.

### Legal and professional services

Possible types include `LegalService`, `Attorney`, `AccountingService`, and
supported professional subtypes. Confirm licenses, people, offices, and
jurisdictions on-site. Do not infer professional status.

### Education

Possible types include `EducationalOrganization`, `School`, `CollegeOrUniversity`,
`Course`, and `LearningResource`. Distinguish an institution, one program, and
one course. Include course/provider data only where the page provides it.

### Nonprofit

`NGO` may be appropriate when the site explicitly supports that identity.
Mission, tax status, service area, and donation claims must come from the site.
Do not infer legal nonprofit status from a Donate button.

### Restaurant and hospitality

Possible types include `Restaurant`, lodging subtypes, `Menu`, and `FoodEvent`.
Use published address, cuisine, reservation, menu, and hours data. Ensure
multiple locations remain separate.

### Ecommerce

Prefer product/catalog plugin output. Improve `Product`, `Offer`, Organization
subtype such as `OnlineStore`, and policy data only from the current storefront.
Do not maintain duplicate prices in theme PHP when commerce data is dynamic.

### Local and home services

Use the closest supported `LocalBusiness` subtype only with published business
identity. Service areas, hours, locations, and phones must be explicit. A service
area is not a physical address.

### B2B and manufacturing

`Organization`, `Corporation`, `Product`, and `Service` are common. Model real
products/services and certifications shown on-site; do not invent offers for
quote-based products.

## Facts that commonly require human or approved off-site data

- latitude/longitude absent from the site;
- official registry identifiers;
- profiles not linked by the site;
- current third-party ratings;
- credentials or accreditations not published;
- legal organization subtype;
- branch-specific hours when pages conflict.

List these as unresolved rather than filling them speculatively.
