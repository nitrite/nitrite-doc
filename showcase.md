---
label: Showcase
icon: server
visibility: public
order: 5
---

<!-- ## Applications using Nitrite Database

Submit your application below to showcase it here. -->


## Showcase Your Application

If you are using Nitrite Database in your application, use the submission form below. It opens a pre-filled GitHub issue draft in the `nitrite-doc` repository, which works reliably with this static documentation site and keeps showcase requests easy to review.

<form action="https://github.com/nitrite/nitrite-doc/issues/new" method="get" target="_blank">
	<p>
		<label for="showcase-title"><strong>Submission title</strong></label><br>
		<input id="showcase-title" name="title" type="text" required placeholder="Showcase: Your application name" style="width: 100%; max-width: 42rem; padding: 0.65rem; border: 1px solid #cbd5e1; border-radius: 0.5rem;" />
	</p>
	<p>
		<label for="showcase-body"><strong>Application details</strong></label><br>
		<textarea id="showcase-body" name="body" rows="16" required style="width: 100%; max-width: 42rem; padding: 0.75rem; border: 1px solid #cbd5e1; border-radius: 0.5rem;">## Application name&#10;&#10;## Short description&#10;&#10;## SDK used&#10;- Rust / Java / Kotlin / Flutter&#10;&#10;## Repository or website&#10;- &#10;&#10;## How Nitrite is used&#10;- &#10;&#10;## Screenshots or demo links&#10;- &#10;&#10;## Contact details&#10;- </textarea>
	</p>
	<p>
		<button type="submit" style="padding: 0.75rem 1rem; border: 0; border-radius: 0.5rem; background: #111827; color: #ffffff; cursor: pointer;">Open GitHub Submission Draft</button>
	</p>
</form>

If the form does not open correctly in your browser, use the direct fallback link: [Create a showcase issue](https://github.com/nitrite/nitrite-doc/issues/new).