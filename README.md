# Injecting Scripts into ShadowRoots

Test cases for various methods of injecting scripts into ShadowRoots.

```
<div id="host"></div>
<script type="module">
  let host = document.querySelector('#host');
  let shadow = host.attachShadow({ mode:'closed' });
  let temp = document.createElement('template');
  temp.innerHTML=`<script> console.log(this);`;
  shadow.append(document.importNode(temp.content, true));
</script>
```

The above script would fail because scripts inserted via innerHTML are not evaluated. This is an expected security measure to prevent the arbitrary execution of scripts injected by malicious third parties. But sometimes, you need to bypass those protections to run trusted scripts inside of a ShadowRoot. Presented below are 5 methods to achieve this goal :

- [Declared Templates](./examples/declared.html)
- [Contextual Templates](./examples/contextual.html)
- [Duplicated Scripts](./examples/duped.html)
- [Spoofed Document](./examples/spoofed.html)
- [Scoped Evaluation](./examples/scoped.html)

The first four of these methods do indeed execute scripts inside the ShadowRoot but they all suffer from one major drawback. The scripts are evaluated in the global scope with no access to the ShadowDOM they were injected into.

The fifth method actually allows us to set the scope of our script and gain access to the ShadowRoot. If used irresponsibly though, this technique can expose your component to script injection attacks.

## Advanced Tests

Below are two detailed examples of using the **Scoped Evaluation** method. The first contains a demonstration of why you should never use this method with an open Shadow Root. The second demonstrates a safe use of this technique to evaulate trusted scripts embedded in the ShadowDOM of a Custom Element :

- [Open ShadowRoot Eval (DANGEROUS!)](./examples/dangerous.html)
- [Closed ShadowRoot Eval](./examples/closed.html)

## Further Considerations

Using the **Closed ShadowRoot Eval** method above causes any script element added to the ShadowRoot to be evaluated on every `connectedCallback`. This might not always be ideal. You may want to do a boolean check to only process the scripts the first time the element is connected to the DOM.

```
#initialized=false;

connectedCallback() {
  !this.#initialized && this.#processScripts() && (this.#initialized=true);
}
```

Do not inject untrusted markup into your ShadowRoot while using this technique, any script elements that content might contain will be executed. You will soon be able to use the [Element.setHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/setHTML) method of the upcoming [HTML Sanitizer API](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Sanitizer_API) to import untrusted markup in a way guaranteed to be free of inline script elements. Until then, a safer method would be to specifically target the script element that you want to execute.

```
get template() {
  return `<script id="test">console.log(this);`;
}

connectedCallback() {
  const test = this.shadowRoot.querySelector('#test');
  Function(test.innerHTML).bind(this.shadowRoot)();
}
```

## Discussion

Further discussion on this topic is in https://github.com/WICG/webcomponents/issues/717#issuecomment-1126786185
