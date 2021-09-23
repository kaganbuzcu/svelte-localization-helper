<script>
  import {
    Container,
    Row,
    Col,
    Textarea,
    TextField,
    Button,
    Icon,
  } from "svelte-materialify";
  import { mdiContentCopy } from "@mdi/js";
  import Clipboard from "svelte-clipboard";
  import Highlight from "svelte-highlight";
  import javascript from "svelte-highlight/src/languages/javascript";
  import atomOneDark from "svelte-highlight/src/styles/atom-one-dark";

  let url = "https://salty-spire-28150.herokuapp.com/api/v1/translate";

  let scopeValue = "";
  let keyValue = "";
  let language1 = "";
  let language2 = "";
  let outputLanguage1 = ".";
  let outputLanguage2 = ".";
  let messages = `/* */`;
  let defineMessages = "";

  function handleLanguage2(event) {
    if (event.key === "Enter") {
      if (outputLanguage1 === ".") {
        outputLanguage1 = `"${scopeValue}.${keyValue}": "${language1}",\n`;
      } else {
        outputLanguage1 += `"${scopeValue}.${keyValue}": "${language1}",\n`;
      }
      if (outputLanguage2 === ".") {
        outputLanguage2 = `"${scopeValue}.${keyValue}": "${language2}",\n`;
      } else {
        outputLanguage2 += `"${scopeValue}.${keyValue}": "${language2}",\n`;
      }

      let fileDescription = scopeValue.split(".");
      defineMessages +=
        keyValue +
        ": {\n\t\tid: `${scope}." +
        keyValue +
        "`,\n\t\tdefaultMessage: '" +
        language1 +
        "'\n\t},\n\t";
      messages = `/*
* ${fileDescription[fileDescription.length - 1]} Messages
*
* This contains all the text for the ${
        fileDescription[fileDescription.length - 1]
      } component.
*/
import { defineMessages } from 'react-intl';

export const scope = '${scopeValue}';

export default defineMessages({
  ${defineMessages}
});
      `;

      keyValue = "";
      language1 = "";
      language2 = "";
      document.getElementById("keyInput").focus();
    }
  }

  async function onBlurLanguage1() {
    if (!!language1) {
      const res = await fetch(`${url}?text=${language1}&from=tr&to=en`, {
        method: "GET",
      });

      const data = await res.json();
      language2 = data;
    }
  }
</script>

<Container fluid>
  <Row>
    <Col>
      <h6 class="mb-3 mt-6">Component path</h6>
      <TextField
        bind:value={scopeValue}
        placeholder="projectName.pages.pageName"
        outlined>Scope</TextField
      >
    </Col>
  </Row>

  <Row>
    <Col>
      <h6 class="mb-3 mt-6">Key of value in object</h6>
      <TextField
        bind:value={keyValue}
        id="keyInput"
        placeholder="camelCaseKey"
        outlined>Key</TextField
      >
    </Col>
    <Col>
      <h6 class="mb-3 mt-6">Language 1</h6>
      <TextField
        bind:value={language1}
        on:blur={onBlurLanguage1}
        placeholder="value"
        outlined>Turkish</TextField
      >
    </Col>
    <Col>
      <h6 class="mb-3 mt-6">Language 2</h6>
      <TextField
        bind:value={language2}
        on:keydown={handleLanguage2}
        placeholder="value"
        outlined>English</TextField
      >
    </Col>
  </Row>

  <Row>
    <Col>
      <div class="copy-button-wrapper">
        <h6 class="mb-3 mt-6">Global translations for language 1</h6>
        <Clipboard
          text={outputLanguage1}
          let:copy
          on:copy={() => {
            console.log("Language #1 Has Copied");
          }}
        >
          <Button on:click={copy} fab size="small">
            <Icon path={mdiContentCopy} />
          </Button>
        </Clipboard>
      </div>
      <Textarea
        value={outputLanguage1}
        on:input={(e) => (outputLanguage1 = e.target.value)}
        rows="15"
        outlined>Global Outputs</Textarea
      >
    </Col>
    <Col>
      <div class="copy-button-wrapper">
        <h6 class="mb-3 mt-6">Global translations for language 2</h6>
        <Clipboard
          text={outputLanguage2}
          let:copy
          on:copy={() => {
            console.log("Language #2 Has Copied");
          }}
        >
          <Button on:click={copy} fab size="small">
            <Icon path={mdiContentCopy} />
          </Button>
        </Clipboard>
      </div>
      <Textarea
        value={outputLanguage2}
        on:input={(e) => (outputLanguage2 = e.target.value)}
        rows="15"
        outlined>Global Outputs</Textarea
      >
    </Col>
  </Row>
  <Row>
    <Col>
      <div class="copy-button-wrapper">
        <h6 class="mb-3 mt-6">messages.js</h6>
        <Clipboard
          text={messages}
          let:copy
          on:copy={() => {
            console.log("messages.js Has Copied");
          }}
        >
          <Button on:click={copy} fab size="small">
            <Icon path={mdiContentCopy} />
          </Button>
        </Clipboard>
      </div>
      <Highlight language={javascript} code={messages} />
    </Col>
  </Row>
</Container>
<svelte:head>
  {@html atomOneDark}
</svelte:head>

<style>
  :global(textarea) {
    font-family: "Consolas";
    font-size: 15px;
  }
  .copy-button-wrapper {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
</style>
