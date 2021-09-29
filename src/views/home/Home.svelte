<script>
  import {
    Container,
    Row,
    Col,
    Textarea,
    TextField,
    Button,
    Icon,
    Tooltip,
    ProgressCircular,
  } from "svelte-materialify";
  import { mdiContentCopy, mdiFileExportOutline } from "@mdi/js";
  import Clipboard from "svelte-clipboard";
  import Highlight from "svelte-highlight";
  import javascript from "svelte-highlight/src/languages/javascript";
  import atomOneDark from "svelte-highlight/src/styles/atom-one-dark";
  import { LanguageSelection } from "../../components";
  import { DataTable } from "../../components";
  import { csvGenerator } from "../../utils/helpers/CsvGenerator";
  import {
    setLocalStorage,
    getLocalStorage,
  } from "../../utils/helpers/LocalStorage";

  let url = "http://localhost:8000/api/v1/translate";

  let languages_details = getLocalStorage("languages_details") || {
    current: { shortName: "tr", name: "Turkish" },
    second: { shortName: "en", name: "English" },
  };
  
  let scopeValue = "";
  let keyValue = "";
  let language1 = "";
  let language2 = "";
  let outputLanguage1 = ".";
  let outputLanguage2 = ".";
  let messages = `/* */`;
  let defineMessages = "";
  let bodyRows = [];
  let tooltipShow = false;
  let translationSuggestion = "";

  function changeLanguages(languages) {
    setLocalStorage("languages_details", languages.detail);
    const { current, second } = languages.detail;
    languages_details = {
      current: { shortName: current.shortName, name: current.name },
      second: { shortName: second.shortName, name: second.name },
    };
  }

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

      bodyRows = [
        ...bodyRows,
        {
          Key: keyValue,
          language1: language1,
          language2: language2,
        },
      ];
      keyValue = "";
      language1 = "";
      language2 = "";
      tooltipShow = false;
      translationSuggestion = "";
      document.getElementById("keyInput").focus();
    } else if (
      event.key === "Tab" &&
      translationSuggestion !== "" &&
      translationSuggestion !== "loading"
    ) {
      language2 = translationSuggestion;
      tooltipShow = false;
      translationSuggestion = "";
      setTimeout(() => {
        document.getElementById("language2Input").focus();
      }, 100);
    }
  }

  async function onBlurLanguage1() {
    if (!!language1) {
      tooltipShow = true;
      translationSuggestion = "loading";
      const res = await fetch(
        `${url}?text=${language1}&from=${languages_details.current.shortName}&to=${languages_details.second.shortName}`,
        {
          method: "GET",
        }
      );

      const data = await res.json();
      translationSuggestion = data;
    }
  }

  function downloadCSVHandler() {
    let tableKeys = Object.keys(bodyRows[0]); //extract key names from first Object
    csvGenerator(bodyRows, tableKeys, tableKeys, "svelte_csv_demo.csv");
  }
</script>

<svelte:head>
  {@html atomOneDark}
</svelte:head>

<LanguageSelection {languages_details} on:languagesChanged={changeLanguages} />

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
      <h6 class="mb-3 mt-6">{languages_details.current.name}</h6>
      <TextField
        bind:value={language1}
        on:blur={onBlurLanguage1}
        placeholder="value"
        outlined>Current Language</TextField
      >
    </Col>
    <Col>
      <h6 class="mb-3 mt-6">{languages_details.second.name}</h6>
      <TextField
        bind:value={language2}
        on:keydown={handleLanguage2}
        id="language2Input"
        placeholder="value"
        outlined>Second Language</TextField
      >
      <Tooltip bind:active={tooltipShow}>
        {#if translationSuggestion === "loading"}
          <ProgressCircular indeterminate color="indigo" />
        {:else}
          <span>{translationSuggestion}</span>
        {/if}
      </Tooltip>
    </Col>
  </Row>

  <Row>
    <Col>
      <div class="copy-button-wrapper">
        <h6 class="mb-3 mt-6">
          Global translations for {languages_details.current.name}
        </h6>
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
        <h6 class="mb-3 mt-6">
          Global translations for {languages_details.second.name}
        </h6>
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
    <Col>
      <div class="copy-button-wrapper">
        <h6 class="mb-3 mt-6">CSV Output</h6>
        <Button on:click={downloadCSVHandler} fab size="small">
          <Icon path={mdiFileExportOutline} />
        </Button>
      </div>
      <DataTable headCells={["Key", "language1", "language2"]} {bodyRows} />
    </Col>
  </Row>
</Container>

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
