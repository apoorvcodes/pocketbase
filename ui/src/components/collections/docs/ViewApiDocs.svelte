<script>
    import { Collection } from "pocketbase";
    import ApiClient from "@/utils/ApiClient";
    import CommonHelper from "@/utils/CommonHelper";
    import CodeBlock from "@/components/base/CodeBlock.svelte";

    export let collection = new Collection();

    let responseTab = 200;
    let sdkTab = "JavaScript";
    let responses = [];
    let sdkExamples = [];

    $: adminsOnly = collection?.viewRule === null;

    $: if (collection?.id) {
        responses.push({
            code: 200,
            body: JSON.stringify(CommonHelper.dummyCollectionRecord(collection), null, 2),
        });

        if (adminsOnly) {
            responses.push({
                code: 403,
                body: `
                    {
                      "code": 403,
                      "message": "Only admins can access this action.",
                      "data": {}
                    }
                `,
            });
        }

        responses.push({
            code: 404,
            body: `
                {
                  "code": 404,
                  "message": "The requested resource wasn't found.",
                  "data": {}
                }
            `,
        });
    }

    $: sdkExamples = [
        {
            lang: "JavaScript",
            code: `
                import PocketBase from 'pocketbase';

                const client = new PocketBase("${ApiClient.baseUrl}");

                client.Records.getOne("${collection?.name}", "RECORD_ID")
                    .then(function (record) {
                        // success...
                    }).catch(function (error) {
                        // error...
                    });
            `,
        },
    ];
</script>

<div class="alert alert-info">
    <strong class="label label-primary">GET</strong>
    <div class="content">
        <p>
            /api/collections/<strong>{collection.name}</strong>/records/<strong>:id</strong>
        </p>
    </div>
    {#if adminsOnly}
        <p class="txt-hint txt-sm txt-right">Requires <code>Authorization: Admin TOKEN</code> header</p>
    {/if}
</div>

<div class="content m-b-base">
    <p>Fetch a single <strong>{collection.name}</strong> record.</p>
</div>

<div class="section-title">Client SDKs example</div>
<div class="tabs m-b-lg">
    <div class="tabs-header compact left">
        {#each sdkExamples as example (example.lang)}
            <button
                class="tab-item"
                class:active={sdkTab === example.lang}
                on:click={() => (sdkTab = example.lang)}
            >
                {example.lang}
            </button>
        {/each}
    </div>
    <div class="tabs-content">
        {#each sdkExamples as example (example.lang)}
            <div class="tab-item" class:active={sdkTab === example.lang}>
                <CodeBlock content={example.code} />
            </div>
        {/each}
    </div>
</div>

<div class="section-title">Path Parameters</div>
<table class="table-compact table-border m-b-lg">
    <thead>
        <tr>
            <th>Param</th>
            <th>Type</th>
            <th width="60%">Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>
                <span class="label">String</span>
            </td>
            <td>ID of the record to view.</td>
        </tr>
    </tbody>
</table>

<div class="section-title">Query parameters</div>
<table class="table-compact table-border m-b-lg">
    <thead>
        <tr>
            <th>Param</th>
            <th>Type</th>
            <th width="60%">Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>expand</td>
            <td>
                <span class="label">String</span>
            </td>
            <td>
                Auto expand nested record relations. Ex.:
                <CodeBlock
                    content={`
                        ?expand=rel1,rel2.subrel21.subrel22
                    `}
                />
                Supports up to 6-levels depth nested relations expansion. <br />
                The expanded relations will be appended to the record under the
                <code>@expand</code> property (eg. <code>{`"@expand": {"rel1": {...}, ...}`}</code>). Only the
                relations that the user has permissions to <strong>view</strong> will be expanded.
            </td>
        </tr>
    </tbody>
</table>

<div class="section-title">Responses</div>
<div class="tabs">
    <div class="tabs-header compact left">
        {#each responses as response (response.code)}
            <button
                class="tab-item"
                class:active={responseTab === response.code}
                on:click={() => (responseTab = response.code)}
            >
                {response.code}
            </button>
        {/each}
    </div>
    <div class="tabs-content">
        {#each responses as response (response.code)}
            <div class="tab-item" class:active={responseTab === response.code}>
                <CodeBlock content={response.body} />
            </div>
        {/each}
    </div>
</div>
