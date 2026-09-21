# WordPress schema code patterns

Adapt these patterns to the client and page. They are not copy/paste defaults:
replace example URLs, function names, types, IDs, and properties with evidence
from the target site.

Official references:

- [Yoast Schema API](https://developer.yoast.com/features/schema/api/)
- [Yoast integration guidelines](https://developer.yoast.com/features/schema/integration-guidelines/)
- [AIOSEO `aioseo_schema_output`](https://aioseo.com/docs/aioseo_schema_output/)
- [Google structured data introduction](https://developers.google.com/search/docs/appearance/structured-data/intro-structured-data)
- [Google structured data policies](https://developers.google.com/search/docs/appearance/structured-data/sd-policies)

## File placement

Use one child-theme file:

```text
functions/schema.php
```

Match the child theme’s existing include convention. A guarded example:

```php
$schema_file = get_stylesheet_directory() . '/functions/schema.php';

if ( file_exists( $schema_file ) ) {
	require_once $schema_file;
}
```

Do not add the include if `schema.php` is already loaded indirectly. Do not edit
the parent Divi theme or an SEO plugin.

## Shared targeting principles

- A numeric `is_page( 123 )` condition is stable inside one WordPress database.
- A canonical URL condition is clear across environments but must account for
  environment domains. Choose intentionally.
- Do not target by page title.
- Normalize only for comparison with `untrailingslashit()`.
- Return the original graph immediately for every non-target page.
- Find existing nodes by stable `@id` first; use `@type` only when the ID is not
  available and the node is unambiguous.
- An `@type` may be a string or an array. Handle both shapes.

Helper for type checks when needed:

```php
/**
 * Determine whether a schema node includes a type.
 *
 * @param array  $node Schema graph node.
 * @param string $type Schema.org type.
 * @return bool
 */
function client_schema_node_has_type( $node, $type ) {
	if ( empty( $node['@type'] ) ) {
		return false;
	}

	$types = (array) $node['@type'];

	return in_array( $type, $types, true );
}
```

Use a client-specific prefix in real projects; `client_` is only a placeholder.

## Yoast: merge or append through the graph

For a page-specific enhancement, `wpseo_schema_graph` provides the completed
graph before output. Preserve all existing nodes and return the graph.

```php
<?php
/**
 * Add evidence-backed Service data to one target page's Yoast graph.
 *
 * The values in this example must be replaced with content visible on the
 * target site.
 *
 * @param array  $graph   Yoast schema graph.
 * @param object $context Yoast meta tags context.
 * @return array
 */
function client_schema_enhance_target_service( $graph, $context ) {
	$target_url = 'https://example.com/services/example-service/';
	$canonical  = isset( $context->canonical ) ? $context->canonical : '';

	if ( untrailingslashit( $canonical ) !== untrailingslashit( $target_url ) ) {
		return $graph;
	}

	$service_id      = trailingslashit( $target_url ) . '#service';
	$web_page_id     = isset( $context->main_schema_id ) ? $context->main_schema_id : '';
	$organization_id = '';

	foreach ( $graph as $node ) {
		if ( isset( $node['@id'] ) && $service_id === $node['@id'] ) {
			return $graph;
		}

		if (
			client_schema_node_has_type( $node, 'WebSite' ) &&
			isset( $node['publisher']['@id'] )
		) {
			$organization_id = $node['publisher']['@id'];
		}
	}

	$service = array(
		'@type'            => 'Service',
		'@id'              => esc_url_raw( $service_id ),
		'name'             => sanitize_text_field( 'Example Service' ),
		'url'              => esc_url_raw( $target_url ),
	);

	if ( $web_page_id ) {
		$service['mainEntityOfPage'] = array(
			'@id' => esc_url_raw( $web_page_id ),
		);
	}

	if ( $organization_id ) {
		$service['provider'] = array(
			'@id' => esc_url_raw( $organization_id ),
		);
	}

	$graph[] = $service;

	return $graph;
}
add_filter( 'wpseo_schema_graph', 'client_schema_enhance_target_service', 11, 2 );
```

Important:

- Do not assume Yoast’s Organization ID. Discover it from the existing graph or
  a Yoast context reference and reuse that exact value.
- Use `$context->main_schema_id` or the live WebPage node for the WebPage `@id`.
  It may differ from the canonical URL and its trailing-slash form.
- If the entity is supporting rather than primary, connect it from the WebPage
  with a type-appropriate property such as `about` or `mentions`. Use `hasPart`
  only when the value is a `CreativeWork` that is genuinely part of the page.
  `isPartOf` is also for `CreativeWork` values and is not a generic
  entity-to-page relationship.
- To enrich one Yoast-owned type rather than add an entity, use the typed filter
  such as `wpseo_schema_organization` and retain the page scope.
- For a reusable custom generator across many pages, use
  `wpseo_schema_graph_pieces` with a class extending Yoast’s
  `Abstract_Schema_Piece`. That is usually excessive for one target page.

## Yoast: merge an existing node

```php
foreach ( $graph as $index => $node ) {
	if ( isset( $node['@id'] ) && $organization_id === $node['@id'] ) {
		$graph[ $index ]['sameAs'] = array_values(
			array_unique(
				array_merge(
					isset( $node['sameAs'] ) ? (array) $node['sameAs'] : array(),
					$verified_same_as_urls
				)
			)
		);
		break;
	}
}
```

Every URL in `$verified_same_as_urls` must be visibly linked or otherwise
supported by the provided site, unless external research was explicitly allowed
and sourced.

## AIOSEO: merge or append through `@graph`

The `aioseo_schema_output` filter receives AIOSEO’s graph entries. Gate the
change to one page and preserve the array.

```php
<?php
/**
 * Add evidence-backed Service data to one target page's AIOSEO graph.
 *
 * @param array $graphs AIOSEO schema graph nodes.
 * @return array
 */
function client_schema_enhance_target_service( $graphs ) {
	if ( ! is_page( 123 ) ) {
		return $graphs;
	}

	$target_url      = get_permalink( 123 );
	$service_id      = trailingslashit( $target_url ) . '#service';
	$web_page_id     = '';
	$organization_id = '';

	foreach ( $graphs as $node ) {
		if ( isset( $node['@id'] ) && $service_id === $node['@id'] ) {
			return $graphs;
		}

		if (
			client_schema_node_has_type( $node, 'WebPage' ) &&
			isset( $node['@id'] )
		) {
			$web_page_id = $node['@id'];
		}

		if (
			client_schema_node_has_type( $node, 'WebSite' ) &&
			isset( $node['publisher']['@id'] )
		) {
			$organization_id = $node['publisher']['@id'];
		}
	}

	$service = array(
		'@type'            => 'Service',
		'@id'              => esc_url_raw( $service_id ),
		'name'             => sanitize_text_field( 'Example Service' ),
		'url'              => esc_url_raw( $target_url ),
	);

	if ( $web_page_id ) {
		$service['mainEntityOfPage'] = array(
			'@id' => esc_url_raw( $web_page_id ),
		);
	}

	if ( $organization_id ) {
		$service['provider'] = array(
			'@id' => esc_url_raw( $organization_id ),
		);
	}

	$graphs[] = $service;

	return $graphs;
}
add_filter( 'aioseo_schema_output', 'client_schema_enhance_target_service' );
```

Read the live AIOSEO graph and use its real Organization and WebPage IDs. Do not
assume they match the example.

## FAQ extraction

Only emit `FAQPage` when each question and answer is visible to visitors on the
target page. Divi toggle/accordion content may qualify when it is present in the
rendered HTML and accessible to users.

First inspect the existing graph. Yoast FAQ blocks can add `FAQPage` to the
WebPage node’s `@type` and emit separate `Question` nodes instead of creating a
standalone FAQPage node. Preserve and extend that native shape when present; do
not add a competing FAQPage node or duplicate questions.

Prefer structured source data when the theme already has it. If values must be
written in PHP, copy their meaning faithfully and strip unsupported markup:

```php
$answer = wp_strip_all_tags( $visible_answer_html );
```

Do not add FAQ schema to unrelated pages, site-wide footers, or content that the
visitor cannot access. Do not promise FAQ rich results; Google eligibility and
display are separate from Schema.org validity.

## Values and output safety

- URLs: `esc_url_raw()` before placing them in arrays.
- Plain text: use trusted WordPress fields and sanitize with
  `sanitize_text_field()` or `wp_strip_all_tags()` as appropriate.
- Multi-line text: `sanitize_textarea_field()` when line breaks matter.
- Never manually concatenate JSON. Return PHP arrays and let the SEO plugin use
  `wp_json_encode()`.
- Do not HTML-escape values with `esc_html()` inside data arrays; JSON encoding
  is the output boundary. Escape any separate HTML output normally.
- Avoid dynamic requests during page rendering. Resolve on-site facts during
  implementation or use local WordPress data.

## Validation checks

Before handoff:

1. `php -l functions/schema.php`
2. Lint `functions.php` if its include changed.
3. Confirm non-target pages return untouched graphs.
4. Confirm no duplicate `@id`.
5. Confirm references point to nodes that exist in the graph.
6. Compare local expected output with deployed live output.
7. Open the target through Schema Markup Validator after deployment.
