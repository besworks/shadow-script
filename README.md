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

The above script would fail because innerHTML creates a new DocumentFragment without a script handling object. This is an expected security measure to prevent the arbitrary execution of scripts injected by malicious third parties. But sometimes, you need to bypass those protections to run trusted scripts inside of a ShadowRoot. Presented below ares 5 methods achieve this goal :

- [Declared Templates](./examples/declared.html)
- [Contextual Templates](./examples/contextual.html)
- [Duplicated Scripts](./examples/duped.html)
- [Spoofed Document](./examples/spoofed.html)
- [Scoped Evaluation](./examples/scoped.html)

The first four of these methods do indeed execute scripts inside the ShadowRoot but they all suffer from one major drawback. The scripts are evaluated in the global scope with no access to the ShadowDOM they were injected into.

The fifth method actually allows us to set the scope of our script and gain access to the ShadowRoot.

## Advanced Tests

Below are two detailed examples of using the **Scoped Evaluation** method. The first contains a demonstration of why you should never use this method with an open Shadow Root. The second demonstrates a safe use of this technique to evaulate trusted scripts embedded in the ShadowDOM of a Custom Element :

- [Open ShadowRoot Eval (DANGEROUS!)](./examples/dangerous.html)
- [Closed ShadowRoot Eval](./examples/closed.html)
