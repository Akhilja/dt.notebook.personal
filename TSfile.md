import { settingsObjectsClient } from "@dynatrace-sdk/client-classic-environment-v2";
import { getEnvironmentId } from "@dynatrace-sdk/app-environment";

export default async function () {
  let mz = await getMZ();
  console.log("Returned Management Zones:", mz);
  return mz;
}

async function getMZ() {
  let config = {
    fields: ["value"], // note: should be plural "fields"
    schemaIds: ["builtin:management-zones"], // array, not string
    pageSize: 500
  };

//  console.log("Fetching MZs with config:", config);

  const result = await settingsObjectsClient.getSettingsObjects(config);

//  console.log("Raw API result:", result);

  return result.items || [];
}
