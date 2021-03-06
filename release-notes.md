---

copyright:
  years: 2019, 2020
lastupdated: "2020-06-17"


keywords: release note, latest changes, Hyperledger Fabric

subcollection: blockchain-sw-25

---

{:note: .note}
{:important: .important}
{:tip: .tip}
{:shortdesc: .shortdesc}
{:pre: .pre}
{:external: target="_blank" .external}

# Release notes
{: #release-notes-saas-20}

<div style="background-color: #f4f4f4; padding-left: 20px; border-bottom: 2px solid #0f62fe; padding-top: 12px; padding-bottom: 4px; margin-bottom: 16px;">
  <p style="line-height: 10px;">
    <strong>Running a different version of IBM Blockchain Platform?</strong> Switch to version
    <a href="https://cloud.ibm.com/docs/blockchain-sw?topic=blockchain-sw-release-notes-saas-20">2.1.2</a>,
    <a href="https://cloud.ibm.com/docs/blockchain-sw-213?topic=blockchain-sw-213-release-notes-saas-20">2.1.3</a>
    </p>
</div>


Use these release notes that are grouped by date to learn about the latest changes to {{site.data.keyword.blockchainfull}} Platform 2.5.
{:shortdesc}

See [Installing patches](/docs/blockchain-sw-25?topic=blockchain-sw-25-console-icp-manage#ibp-console-manage-patch) for instructions on how to apply patches to your existing nodes.

## 18 June 2020
{: #06-18-2020}

**Peer and ordering node patch 1.4.7-0**

**{{site.data.keyword.IBM_notm}} considers this a critical patch that you should apply at your nearest opportunity.** Not applying this patch risks being susceptible to a data integrity issue if you experience a CouchDB or underlying storage crash on your peer. This patch updates the CouchDB `delayed_commits` configuration property to false. The prior setting could cause the peer's CouchDB database to be in an inconsistent state in the event of a CouchDB or underlying storage system crash. The new setting ensures that a peer will recover from crashes in a consistent state. As always, it is recommended to utilize an endorsement policy that requires multiple peers to endorse a transaction, to avoid an inconsistency from a single peer from impacting the overall blockchain network state.  

**To be certain that your peers have no database corruption**, you should reprovision your peers.

Miscellaneous bug fixes and security patches.  

### Fabric peer and ordering node images
{: #06-18-2020-images}

The platform introduces the capability to deploy new peer and ordering nodes based on either Hyperledger Fabric v1.4 or v2.x. Deploying peer and ordering nodes with the latest Fabric images is recommended to ensure that you have access to current Fabric fixes and features. It is not currently possible to migrate existing nodes to Fabric v2 images.

### Elimination of Docker daemon dependency
{: #06-18-2020-docker}

Leveraging the Fabric v2 **external chaincode launcher** capability, when you deploy a peer based on the Fabric v2.1.1 image, smart contracts are deployed into their own pod rather than inside a container on the peer pod.

### Multizone-capable storage
{: #06-18-2020-Multizone}

If your Kubernetes cluster is configured to use multizone-capable storage, new peer and ordering nodes can be deployed that leverage multizone storage, effectively extending their high availability across cluster zones.See [Multizone-capable Storage](/docs/blockchain-sw-25?topic=blockchain-sw-25-ibp-console-ha##ibp-console-ha-multi-zone-storage) for more information.

