---
title: Consumer Version Selectors
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

## Overview

Consumer version selectors are the way to configure which pacts the provider verifies. A list of selector objects is provided, allowing flexibity in which pacts to verify and enabling you to build resilient and comprehensive verification setups.

<a name="golden-rule"/>

:::info Default selectors
The default selectors find pacts that match any of the following criteria:

1. latest version on the main branch.
1. all versions that have been deployed.
1. all versions that are currently marked as released.
  
The default selectors are a good starting point, before moving to the [recommended selectors](#recommended).
:::

## Properties

A consumer version selector has the following properties:

- `mainBranch`: if the key is specified, can only be set to `true`. Return the pacts for the configured `mainBranch` of each consumer. Use of this selector requires that the consumer has configured the `mainBranch` property, and has set a branch name when publishing the pacts. As of October 2021, this is not yet supported in all Pact client libraries.

- `branch`: the branch name of the consumer versions to get the pacts for. Use of this selector requires that the consumer has configured a branch name when publishing the pacts. As of October 2021, this is not yet supported in all Pact client libraries.

- `fallbackBranch`: the name of the branch to fallback to if the specified `branch` does not exist. Use of this property is discouraged as it may allow a pact to pass on a feature branch while breaking backwards compatibility with the main branch, which is generally not desired. It is better to use two separate consumer version selectors, one with the main branch name, and one with the feature branch name, rather than use this property. As of October 2021, this is not yet supported in all Pact client libraries.

- `tag`: the tag name(s) of the consumer versions to get the pacts for. *This field is still supported but it is recommended to use the `branch` in preference now.*

- `fallbackTag`: the name of the tag to fallback to if the specified `tag` does not exist. *This field is still supported but it is recommended to use the `fallbackBranch` in preference now.*

- `deployed`: if the key is specified, can only be set to `true`. Returns the pacts for all versions of the consumer that are currently deployed to any environment. Use of this selector requires that the deployment of the consumer application is recorded in the Pact Broker using the `pact-broker record-deployment` CLI. As of October 2021, this is not yet supported in all Pact client libraries.

- `released`: if the key is specified, can only be set to `true`. Returns the pacts for all versions of the consumer that are released and currently supported in any environment. Use of this selector requires that the deployment of the consumer application is recorded in the Pact Broker using the `pact-broker record-release` CLI. As of October 2021, this is not yet supported in all Pact client libraries.

- `deployedOrReleased`: if the key is specified, can only be set to `true`. Returns the pacts for all versions of the consumer that are currently deployed or released and currently supported in any environment. Use of this selector requires that the deployment of the consumer application is recorded in the Pact Broker using the `pact-broker record-deployment` or `record-release` CLI. As of October 2021, this is not yet supported in all Pact client libraries.

- `environment`: the name of the environment containing the consumer versions for which to return the pacts. Used to further qualify `{ "deployed": true }` or `{ "released": true }`. Normally, this would not be needed, as it is recommended to verify the pacts for all currently deployed/currently supported released versions. As of October 2021, this is not yet supported in all Pact client libraries.

- `latest`: true. Used in conjuction with the `tag` property. If a `tag` is specified, and `latest` is `true`, then the latest pact for each of the consumers with that tag will be returned. If a `tag` is specified and the latest flag is *not* set to `true`, *all* the pacts with the specified tag will be returned. (This might seem a bit weird, but it's done this way to match the syntax used for the matrix query params. See <https://docs.pact.io/selectors>).

- `consumer`: allows a selector to only be applied to a certain consumer. Can be specified with any of the above properties.

## Examples

### Recommended

These are the recommended selectors that will cover the majority of workflows.

- `{ "mainBranch": true }` - the latest version from the main branch of each consumer, as specified by the consumer's `mainBranch` property.
- `{ "branch": "<branch>" }` - the latest version from a particular branch of each consumer.
- `{ "deployedOrReleased": true }` - all the currently deployed and currently released and supported versions of each consumer.
- `{ "matchingBranch": true }` - the latest version from any branch of the consumer that has the same name as the current branch of the provider. Used for coordinated development between consumer and provider teams using matching feature branch names.

### Advanced

This is not an exhaustive list, but shows most of the usecases.

- `{ "branch": "<branch>", consumer: "<consumer>" }` - the latest version from a particular branch of a particular consumer.
- `{ "branch": "<branch>", "fallbackBranch": "<branch>" }` - the latest version from a particular branch of the consumer, falling back to the fallbackBranch if none is found from the specified branch.
- `{ "deployed": true, environment: "<environment>" }` - any versions currently deployed to the specified environment
- `{ "released": true, environment: "<environment>" }` - any versions currently released and supported in the specified environment
- `{ "environment": "<environment>" }` - any versions currently deployed or released and supported in the specified environment
- `{ "tag": "<tag>" }`- all versions with the specified tag
- `{ "tag": "<tag>", "latest": true }`- the latest version for each consumer with the specified tag
- `{ "tag": "<tag>", "latest": true, "fallbackTag": "<tag>" }`- the latest version for each consumer with the specified tag, falling back to the fallbackTag if non is found with the specified tag.
- `{ "tag": "<tag>", "latest": true, "consumer": "<consumer>" }`- the latest version for a specified consumer with the specified tag
- `{ "latest": true }` - the latest version for each consumer. NOT RECOMMENDED as it suffers from race conditions when pacts are published from multiple branches.

## Deduplication

The Pact Broker API for retrieving pacts by selectors deduplicates the pacts based on their *content*. This means that if the same content was published in multiple selected pacts, the verification for that content will only need to run once. This is quite common when there hasn't been a change to a pact for a while, and the same pact content is present in development, test and production pacts.

## Docs

You can checkout code-snippets below, but here are some links to either documentation, or source code, from the respective languages, around their use of consumer version selectors. Ideally all languages should support raw json version selectors, to allow for extensiblity in the future.

- JavaScript
  - <https://docs.pact.io/implementation_guides/javascript/docs/provider#verification-options>
  - <https://github.com/pact-foundation/pact-js-core/blob/master/src/verifier/types.ts#L3>
- Java
  - <https://docs.pact.io/implementation_guides/jvm/provider/junit#selecting-the-pacts-to-verify-with-consumer-version-selectors-4314>
- Gradle
  - <https://docs.pact.io/implementation_guides/jvm/provider/gradle#using-consumer-version-selectors-4310>
- Ruby
  - <https://docs.pact.io/implementation_guides/ruby/verifying_pacts#fetching-pacts-from-a-pact-broker>
  - <https://github.com/pact-foundation/pact-ruby/blob/master/lib/pact/pact_broker/pact_selection_description.rb>
- Python - Takes raw json consumer_version_selectors
  - <https://github.com/pact-foundation/pact-python/blob/89733d6470cfa4b57102438c464f053093ddd858/pact/verifier.py#L109>
- C#
  - <https://github.com/pact-foundation/pact-net/blob/master/docs/upgrading-to-4.md#provider-tests>
  - <https://github.com/pact-foundation/pact-net/blob/master/src/PactNet.Abstractions/Verifier/ConsumerVersionSelector.cs>
- Golang
  - <https://github.com/pact-foundation/pact-go/blob/2.x.x/docs/provider.md#selecting-pacts-to-verify>
  - <https://github.com/pact-foundation/pact-go/blob/master/provider/consumer_version_selector.go>

## Code examples with branches

### Verifying the latest development, test and master pacts

This is the most common use case.

- We recommend using `deployedOrReleased` to return the pacts for all versions of the consumer that are currently deployed or released and currently supported in any environment, this allows the provider to not know that detail and let the Pact broker take care of it for you)
  - alternatively you can target just `deployed` or just `released` consumers across `all` environments , or consumers only in specified `environments`.
- We recommend using `mainBranch` to return the pacts associated with any consumers configured `mainBranch` property (your consumer may have the branch `main`, `master` or `develop`, this allows the provider to not know that detail and let the Pact broker take care of it for you)

<Tabs
groupId="sdk-choice"
defaultValue="javascript"
values={[
{label: 'Javascript', value: 'javascript', },
{label: 'Java', value: 'java', },
{label: 'Gradle', value: 'gradle', },
{label: 'Ruby', value: 'ruby', },
{label: 'Python', value: 'python', },
{label: 'C#', value: 'c#', },
{label: 'Golang', value: 'golang', }
]
}>
<TabItem value="javascript">

```js
const verificationOptions = {
  // ....
  consumerVersionSelectors: [
    {
      mainBranch: true, 
    },
    {
      deployedOrReleased: true,
    }
  ],
};
```

  </TabItem>

  <TabItem value="java">

```java
  @au.com.dius.pact.provider.junitsupport.loader.PactBrokerConsumerVersionSelectors
  public static SelectorBuilder consumerVersionSelectors() {
    return new SelectorBuilder()
      .mainBranch(); 
      .deployedOrReleased(); 
  }

```

  </TabItem>

  <TabItem value="gradle">

```groovy

pact {
  serviceProviders {
    'Your Service' {
      providerVersion = { '1.2.3' }

      fromPactBroker {
        withSelectors {
            mainBranch() 
            deployedOrReleased()
        }
      }
    }
  }
}
```

  </TabItem>

  <TabItem value="ruby">

```ruby
Pact.service_provider "Your provider" do
  honours_pacts_from_pact_broker do
    pact_broker_base_url "..."
    consumer_version_selectors [
          { mainBranch: true },
          { deployedOrReleased: true }
        ]
  end
end
```

  </TabItem>

  <TabItem value="python">

```python
verifier = Verifier(
    provider="Your provider",
    broker_url="...",
    consumer_version_selectors=[
        {"mainBranch": True},
        {"deployedOrReleased": True}
    ],
    # ...
)
```

  </TabItem>

  <TabItem value="c#">

```csharp
    IPactVerifier verifier = new PactVerifier(config); 
    verifier.ServiceProvider("My Provider", this.fixture.ServerUri)
        .WithPactBrokerSource(new Uri("https://broker.example.org"), options =>
        {
            options.ConsumerVersionSelectors(new ConsumerVersionSelector { MainBranch = true }, 
                                                  new ConsumerVersionSelector { DeployedOrReleased = true }
                                                  )
                    .PublishResults(version, results =>
                    {
                        results.ProviderBranch(branch)
                              .BuildUri(new Uri(buildUri));
                    });
        })
        .WithProviderStateUrl(new Uri(this.fixture.ServerUri, "/provider-states"))
        .Verify();       
```

  </TabItem>

  <TabItem value="golang">

```golang
 pact.VerifyProvider(t, provider.VerifyRequest{
  ConsumerVersionSelectors: []provider.Selector{
   &provider.ConsumerVersionSelector{
    MainBranch:      true,  
   },
   &provider.ConsumerVersionSelector{
    DeployedOrReleased:      true,
   }
  },
        // ...
 })
```

  </TabItem>

</Tabs>

### Using a matching branch for coordinated branch development

Dynamically determine the current branch of the provider, see if there is a matching pact for that branch.

`matchingBranch` -  Returns the pacts for providers configured branch property in the verification task. Normally covered by contract_requiring_verification_published event + corresponding webhoo

<Tabs
groupId="sdk-choice"
defaultValue="javascript"
values={[
{label: 'Javascript', value: 'javascript', },
{label: 'Java', value: 'java', },
{label: 'Gradle', value: 'gradle', },
{label: 'Ruby', value: 'ruby', },
{label: 'Python', value: 'python', },
{label: 'C#', value: 'c#', },
{label: 'Golang', value: 'golang', }
]
}>
<TabItem value="javascript">

```js
const verificationOptions = {
  //...
  consumerVersionSelectors: [
    {
      matchingBranch: true,
    },
    {
      mainBranch: true, 
    },
    {
      deployedOrReleased: true,
    },
  ],
};
```

  </TabItem>

  <TabItem value="java">

```java
// Requires Pact-JVM 4.1.10 or later
  @au.com.dius.pact.provider.junitsupport.loader.PactBrokerConsumerVersionSelectors
  public static SelectorBuilder consumerVersionSelectors() {
    return new SelectorBuilder()
      .matchingBranch();
      .mainBranch(); 
      .deployedOrReleased(); 
  }
```

  </TabItem>

  <TabItem value="gradle">

```groovy
// Requires Pact-JVM 4.1.10 or later
pact {
  serviceProviders {
    'Your Service' {
      providerVersion = { '1.2.3' }

      fromPactBroker {
        withSelectors {
            matchingBranch()
            mainBranch() 
            deployedOrReleased()
        }
      }
    }
  }
}
```

  </TabItem>

  <TabItem value="ruby">

```ruby
Pact.service_provider "Your provider" do
  honours_pacts_from_pact_broker do
    pact_broker_base_url "..."
    consumer_version_selectors [
          { matchingBranch: true }
          { mainBranch: true }
          { deployedOrReleased: true }
        ]
  end
end
```

  </TabItem>

  <TabItem value="python">

```python
verifier = Verifier(
    provider="Your provider",
    broker_url="...",
    consumer_version_selectors=[
        {"matchingBranch": True},
        {"mainBranch": True},
        {"deployedOrReleased": True}
    ],
    # ...
)
```

  </TabItem>

  <TabItem value="c#">

```csharp
    IPactVerifier verifier = new PactVerifier(config); 
    verifier.ServiceProvider("My Provider", this.fixture.ServerUri)
        .WithPactBrokerSource(new Uri("https://broker.example.org"), options =>
        {
            options.ConsumerVersionSelectors(new ConsumerVersionSelector { MatchingBranch = true }
                                                  new ConsumerVersionSelector { DeployedOrReleased = true }
                                                  new ConsumerVersionSelector { MainBranch = true }, 
                                                  )
                    .PublishResults(version, results =>
                    {
                        results.ProviderBranch(branch)
                              .BuildUri(new Uri(buildUri));
                    });
        })
        .WithProviderStateUrl(new Uri(this.fixture.ServerUri, "/provider-states"))
        .Verify();     
```

  </TabItem>

  <TabItem value="golang">

```golang
 pact.VerifyProvider(t, provider.VerifyRequest{
  ConsumerVersionSelectors: []provider.Selector{
   &provider.ConsumerVersionSelector{
    MatchingBranch:      true,
   },
   &provider.ConsumerVersionSelector{
    MainBranch:      true,  
   },
   &provider.ConsumerVersionSelector{
    DeployedOrReleased:      true,
   },
  },
        // ...
 })
```

  </TabItem>

</Tabs>

### Verifying pacts where the consumer is a mobile application

Verify the pacts for the latest `master` and `test` versions, and all `production` versions of "my-mobile-consumer".

<Tabs
groupId="sdk-choice"
defaultValue="javascript"
values={[
{label: 'Javascript', value: 'javascript', },
{label: 'Java', value: 'java', },
{label: 'Gradle', value: 'gradle', },
{label: 'Ruby', value: 'ruby', },
{label: 'Python', value: 'python', },
{label: 'C#', value: 'c#', },
{label: 'Golang', value: 'golang', }
]
}>
<TabItem value="javascript">

```js
const verificationOptions = {
  // ....
  consumerVersionSelectors: [
    {
      mainBranch: true, 
    },
    {
      deployed: 'test',
    },
    {
      deployedOrReleased: 'production', consumer: 'my-mobile-consumer'
    },
  ],
};
```

  </TabItem>

  <TabItem value="java">

```java
  @au.com.dius.pact.provider.junitsupport.loader.PactBrokerConsumerVersionSelectors
  public static SelectorBuilder consumerVersionSelectors() {
    return new SelectorBuilder()
      .mainBranch(); 
      .deployedTo('test');
      .rawSelectorJson('{ "deployedOrReleased": "production", "consumer": "my-mobile-consumer" }')
  }

```

  </TabItem>

  <TabItem value="gradle">

```groovy

pact {
  serviceProviders {
    'Your Service' {
      providerVersion = { '1.2.3' }

      fromPactBroker {
        withSelectors {
            mainBranch() 
            deployedTo('test')
            rawSelectorJson('{ "deployedOrReleased": "production", "consumer": "my-mobile-consumer" }')
        }
      }
    }
  }
}
```

  </TabItem>

  <TabItem value="ruby">

```ruby
Pact.service_provider "Your provider" do
  honours_pacts_from_pact_broker do
    pact_broker_base_url "..."
    consumer_version_selectors [
          { mainBranch: true },
          { deployed: "test" },
          { deployedOrReleased: "production", consumer: "my-mobile-consumer" } 
        ]
  end
end
```

  </TabItem>

  <TabItem value="python">

```python
verifier = Verifier(
    provider="Your provider",
    broker_url="...",
    consumer_version_selectors=[
        {"mainBranch": True},
        {"deployed": "test"},
        {"deployedOrReleased": "production", consumer: "my-mobile-consumer"}
    ],
    # ...
)
```

  </TabItem>

  <TabItem value="c#">

```csharp
    IPactVerifier verifier = new PactVerifier(config); 
    verifier.ServiceProvider("My Provider", this.fixture.ServerUri)
        .WithPactBrokerSource(new Uri("https://broker.example.org"), options =>
        {
            options.ConsumerVersionSelectors(new ConsumerVersionSelector { MainBranch = true }, 
                                                  new ConsumerVersionSelector { Deployed = true, Environment = "test" },
                                                  new ConsumerVersionSelector { Released = true, Deployed = true, Environment = "production", Consumer: "my-mobile-consumer" }
                                                  )
                    .PublishResults(version, results =>
                    {
                        results.ProviderBranch(branch)
                              .BuildUri(new Uri(buildUri));
                    });
        })
        .WithProviderStateUrl(new Uri(this.fixture.ServerUri, "/provider-states"))
        .Verify();       
```

  </TabItem>

  <TabItem value="golang">

```golang
 pact.VerifyProvider(t, provider.VerifyRequest{
  ConsumerVersionSelectors: []provider.Selector{
   &provider.ConsumerVersionSelector{
    MainBranch:      true,  
   },
   &provider.ConsumerVersionSelector{
    Deployed:         "test", // only the latest deployed version to test
   },
   &provider.ConsumerVersionSelector{
    DeployedOrReleased: "production",
    Consumer:           "my-mobile-consumer"
   },
  },
        // ...
 })
```

  </TabItem>

</Tabs>

### Verifying a pacts where one consumer is a mobile application

Verify the latest `production` version of all consumers, and all `production` versions of "my-mobile-consumer". Note that the pacts are [deduplicated](#deduplication), so despite being included by 2 selectors, the verification of the latest production pact for "my-mobile-consumer" will only run once.

<Tabs
groupId="sdk-choice"
defaultValue="javascript"
values={[
{label: 'Javascript', value: 'javascript', },
{label: 'Java', value: 'java', },
{label: 'Gradle', value: 'gradle', },
{label: 'Ruby', value: 'ruby', },
{label: 'Python', value: 'python', },
{label: 'C#', value: 'c#', },
{label: 'Golang', value: 'golang', }
]
}>
<TabItem value="javascript">

```js
const verificationOptions = {
  // ....
  consumerVersionSelectors: [
    {
      mainBranch: true, 
    },
    {
      deployed: 'test',
    },
    {
      deployed: 'production',
    },
    {
      deployedOrReleased: 'production',
      consumer: 'my-mobile-consumer'
    },
  ],
};
```

  </TabItem>

  <TabItem value="java">

```java
  @au.com.dius.pact.provider.junitsupport.loader.PactBrokerConsumerVersionSelectors
  public static SelectorBuilder consumerVersionSelectors() {
    return new SelectorBuilder()
      .mainBranch(); 
      .deployedTo('test');
      .deployedTo('production');
      .rawSelectorJson('{ "deployedOrReleased": "production", "consumer": "my-mobile-consumer" }')
  }

```

  </TabItem>

  <TabItem value="gradle">

```groovy

pact {
  serviceProviders {
    'Your Service' {
      providerVersion = { '1.2.3' }

      fromPactBroker {
        withSelectors {
            mainBranch() 
            deployedTo('test')
            deployedTo('production')
            rawSelectorJson('{ "deployedOrReleased": "production", "consumer": "my-mobile-consumer" }')
        }
      }
    }
  }
}
```

  </TabItem>

  <TabItem value="ruby">

```ruby
Pact.service_provider "Your provider" do
  honours_pacts_from_pact_broker do
    pact_broker_base_url "..."
    consumer_version_selectors [
          { mainBranch: true },
          { deployed: "test" },
          { deployed: "production" },
          { deployedOrReleased: "production", consumer: "my-mobile-consumer" }
        ]
  end
end
```

  </TabItem>

  <TabItem value="python">

```python
verifier = Verifier(
    provider="Your provider",
    broker_url="...",
    consumer_version_selectors=[
        {"mainBranch": True},
        {"deployed": "test"},
        {"deployed": "production"},
        {"deployedOrReleased": "production", "consumer": "my-mobile-consumer" },
    ],
    # ...
)
```

  </TabItem>

  <TabItem value="c#">

```csharp
    IPactVerifier verifier = new PactVerifier(config); 
    verifier.ServiceProvider("My Provider", this.fixture.ServerUri)
        .WithPactBrokerSource(new Uri("https://broker.example.org"), options =>
        {
            options.ConsumerVersionSelectors(new ConsumerVersionSelector { MainBranch = true }, 
                                                  new ConsumerVersionSelector { Deployed = true, Environment = "test" },
                                                  new ConsumerVersionSelector { Deployed = true, Environment = "production" },
                                                  new ConsumerVersionSelector { Released = true, Deployed = true, Environment = "production", Consumer = "my-mobile-consumer" },
                                                  )
                    .PublishResults(version, results =>
                    {
                        results.ProviderBranch(branch)
                              .BuildUri(new Uri(buildUri));
                    });
        })
        .WithProviderStateUrl(new Uri(this.fixture.ServerUri, "/provider-states"))
        .Verify();       
```

  </TabItem>

  <TabItem value="golang">

```golang
 pact.VerifyProvider(t, provider.VerifyRequest{
  ConsumerVersionSelectors: []provider.Selector{
   &provider.ConsumerVersionSelector{
    MainBranch:      true,  
   },
   &provider.ConsumerVersionSelector{
    Deployed:         "test",
   },
   &provider.ConsumerVersionSelector{
    Deployed:         "production",
   },
   &provider.ConsumerVersionSelector{
    DeployedOrReleased:         "production",
    Consumer:                   "my-mobile-consumer",
   },
  },
        // ...
 })
```

  </TabItem>

</Tabs>

### Verifying the overall latest pact for each consumer

Verifying the overall latest pact for each consumer is syntactically possible, but not recommended, as pacts for different branches of the consumer may overwrite each other as the current latest.

You can do this by setting `latest: true`, however as this is wholly unrecommended, we instead show the minimum ideal requirement, to cover the latest pacts for

- any consumer that is registered with the provider
  - the latest pact for any environment the consumer is `deployed` to
  - the latest pact, for any environment in the consumer is `released` to
  - the latest pact, for the consumer configured `mainBranch`

<Tabs
groupId="sdk-choice"
defaultValue="javascript"
values={[
{label: 'Javascript', value: 'javascript', },
{label: 'Java', value: 'java', },
{label: 'Gradle', value: 'gradle', },
{label: 'Ruby', value: 'ruby', },
{label: 'Python', value: 'python', },
{label: 'C#', value: 'c#', },
{label: 'Golang', value: 'golang', }
]
}>
<TabItem value="javascript">

```js
const verificationOptions = {
  // ....
  consumerVersionSelectors: [
    {
      mainBranch: true, 
    },
    {
      deployedOrReleased: true,
    },
  ],
};
```

  </TabItem>

  <TabItem value="java">

```java
  @au.com.dius.pact.provider.junitsupport.loader.PactBrokerConsumerVersionSelectors
  public static SelectorBuilder consumerVersionSelectors() {
    return new SelectorBuilder()
      .mainBranch(); 
      .deployedOrReleased(); 
  }

```

  </TabItem>

  <TabItem value="gradle">

```groovy

pact {
  serviceProviders {
    'Your Service' {
      providerVersion = { '1.2.3' }

      fromPactBroker {
        withSelectors {
            mainBranch() 
            deployedOrReleased()
        }
      }
    }
  }
}
```

  </TabItem>

  <TabItem value="ruby">

```ruby
Pact.service_provider "Your provider" do
  honours_pacts_from_pact_broker do
    pact_broker_base_url "..."
    consumer_version_selectors [
          { mainBranch: true },
          { deployedOrReleased: true }
        ]
  end
end
```

  </TabItem>

  <TabItem value="python">

```python
verifier = Verifier(
    provider="Your provider",
    broker_url="...",
    consumer_version_selectors=[
        {"mainBranch": True},
        {"deployedOrReleased": True},
    ],
    # ...
)
```

  </TabItem>

  <TabItem value="c#">

```csharp
    IPactVerifier verifier = new PactVerifier(config); 
    verifier.ServiceProvider("My Provider", this.fixture.ServerUri)
        .WithPactBrokerSource(new Uri("https://broker.example.org"), options =>
        {
            options.ConsumerVersionSelectors(new ConsumerVersionSelector { MainBranch = true }, 
                                                  new ConsumerVersionSelector { DeployedOrReleased = true }
                                                  )
                    .PublishResults(version, results =>
                    {
                        results.ProviderBranch(branch)
                              .BuildUri(new Uri(buildUri));
                    });
        })
        .WithProviderStateUrl(new Uri(this.fixture.ServerUri, "/provider-states"))
        .Verify();       
```

  </TabItem>

  <TabItem value="golang">

```golang
 pact.VerifyProvider(t, provider.VerifyRequest{
  ConsumerVersionSelectors: []provider.Selector{
   &provider.ConsumerVersionSelector{
    MainBranch:      true,  
   },
   &provider.ConsumerVersionSelector{
    DeployedOrReleased:      true,
   },
  },
        // ...
 })
```

  </TabItem>

</Tabs>

## Code examples with tags

### Verifying the latest development, test and master pacts

This is the most common use case.

<Tabs
groupId="sdk-choice-tags"
defaultValue="javascript"
values={[
{label: 'Javascript', value: 'javascript', },
{label: 'Java', value: 'java', },
{label: 'Gradle', value: 'gradle', },
{label: 'Ruby', value: 'ruby', },
{label: 'Python', value: 'python', },
{label: 'C#', value: 'c#', },
{label: 'Golang', value: 'golang', }
]
}>
<TabItem value="javascript">

```js
const verificationOptions = {
  // ....
  consumerVersionSelectors: [
    {
      tag: "master",
      latest: true,
    },
    {
      tag: "test",
      latest: true,
    },
    {
      tag: "production",
      latest: true,
    },
  ],
};
```

  </TabItem>

  <TabItem value="java">

```java
@PactBroker(
  host="pactbroker.local",
  port="8080",
  consumerVersionSelectors={
    @VersionSelector(tag = "master"),
    @VersionSelector(tag = "test"),
    @VersionSelector(tag = "production")
  }
)

```

  </TabItem>

  <TabItem value="gradle">

```groovy

pact {
  serviceProviders {
    'Your Service' {
      providerVersion = { '1.2.3' }

      fromPactBroker {
        selectors = latestTags('master', 'test', 'production')
      }
    }
  }
}
```

  </TabItem>

  <TabItem value="ruby">

```ruby
Pact.service_provider "Your provider" do
  honours_pacts_from_pact_broker do
    pact_broker_base_url "..."
    consumer_version_selectors [
          { tag: "main", latest: true },
          { tag: "test", latest: true },
          { tag: "production", latest: true }
        ]
  end
end
```

  </TabItem>

  <TabItem value="python">

```python
verifier = Verifier(
    provider="Your provider",
    broker_url="...",
    consumer_version_selectors=[
        {"tag": "main", "latest": True},
        {"tag": "test", "latest": True},
        {"tag": "production", "latest": True},
    ],
    # ...
)
```

  </TabItem>

  <TabItem value="c#">

```csharp
    var pactVerifier = new PactVerifier(new PactVerifierConfig())
      .ProviderState($"providerServiceBaseUri/provider-states")
      .ServiceProvider("ProviderPacticipantName", "providerServiceBaseUri")
      .HonoursPactWith("ConsumerPacticipantName")
      .PactBroker(
        "http://pact-broker",
        consumerVersionSelectors: new List<VersionTagSelector>
        {
          new VersionTagSelector("master", latest: true),
          new VersionTagSelector("test", latest: true),
          new VersionTagSelector("production", latest: true)
        });
```

  </TabItem>

  <TabItem value="golang">

```golang
 pact.VerifyProvider(t, provider.VerifyRequest{
  ConsumerVersionSelectors: []provider.Selector{
   &provider.ConsumerVersionSelector{
    Tag:         "master",
    Latest:      true,
   },
   &provider.ConsumerVersionSelector{
    Tag:         "test",
    Latest:      true,
   },
   &provider.ConsumerVersionSelector{
    Tag:         "production",
    Latest:      true,
   },
  },
        // ...
 })
```

  </TabItem>

</Tabs>

### Using a fallback tag for coordinated branch development

Dynamically determine the current branch of the provider, see if there is a matching pact for that branch, fallback to the `master` pact if none exists. This approach is not longer recommended - see the note on the `fallbackTag` at the top of the page.

<Tabs
groupId="sdk-choice"
defaultValue="javascript"
values={[
{label: 'Javascript', value: 'javascript', },
{label: 'Java', value: 'java', },
{label: 'Gradle', value: 'gradle', },
{label: 'Ruby', value: 'ruby', },
{label: 'Python', value: 'python', },
{label: 'C#', value: 'c#', },
{label: 'Golang', value: 'golang', }
]
}>
<TabItem value="javascript">

```js
const verificationOptions = {
  //...
  consumerVersionSelectors: [
    {
      tag: process.env.GIT_BRANCH,
      fallbackTag: "master",
      latest: true,
    },
    {
      tag: "test",
      latest: true,
    },
    {
      tag: "production",
      latest: true,
    },
  ],
};
```

  </TabItem>

  <TabItem value="java">

```java
// Requires Pact-JVM 4.1.10 or later
@PactBroker(
  host="pactbroker.local",
  port="8080",
  consumerVersionSelectors={
    @VersionSelector(tag = "${GIT_BRANCH}", fallbackTag = "master"),
    @VersionSelector(tag = "test"),
    @VersionSelector(tag = "production")
  }
)

```

  </TabItem>

  <TabItem value="gradle">

```groovy
// Requires Pact-JVM 4.1.10 or later
pact {
  serviceProviders {
    'Your Service' {
      providerVersion = { '1.2.3' }

      fromPactBroker {
        selectors = latestTags(fallbackTag: 'master', System.getenv('GIT_BRANCH'))
      }
    }
  }
}
```

  </TabItem>

  <TabItem value="ruby">

```ruby
Pact.service_provider "Your provider" do
  honours_pacts_from_pact_broker do
    pact_broker_base_url "..."
    consumer_version_selectors [
        { tag: ENV["GIT_BRANCH"], fallback_tag: "main", latest: true },
        { tag: "test", latest: true },
        { tag: "production", latest: true }
      ]
  end
end
```

  </TabItem>

  <TabItem value="python">

```python
verifier = Verifier(
    provider="Your provider",
    broker_url="...",
    consumer_version_selectors=[
        {"tag": os.environ.get("GIT_BRANCH"), "fallback_tag": "main", "latest": True},
        {"tag": "test", "latest": True},
        {"tag": "production", "latest": True},
    ],
    # ...
)
```

  </TabItem>

  <TabItem value="c#">

```csharp
     var pactVerifier = new PactVerifier(new PactVerifierConfig())
      .ProviderState($"providerServiceBaseUri/provider-states")
      .ServiceProvider("ProviderPacticipantName", "providerServiceBaseUri")
      .HonoursPactWith("ConsumerPacticipantName")
      .PactBroker(
        "http://pact-broker",
        consumerVersionSelectors: new List<VersionTagSelector>
        {
          new VersionTagSelector(Environment.GetEnvironmentVariable("GIT_BRANCH"), fallbackTag: "master", latest: true),
          new VersionTagSelector("test", latest: true),
          new VersionTagSelector("production", latest: true)
        });
```

  </TabItem>

  <TabItem value="golang">

```golang
 pact.VerifyProvider(t, provider.VerifyRequest{
    ConsumerVersionSelectors: []provider.Selector{
   &provider.ConsumerVersionSelector{
    Tag:         os.Getenv("GIT_BRANCH"),
    FallbackTag: "master",
    Latest:      true,
   },
   &provider.ConsumerVersionSelector{
    Tag:    "test",
    Latest: true,
   },
   &provider.ConsumerVersionSelector{
    Tag:    "production",
    Latest: true,
   },
  },
        // ...
 })
```

  </TabItem>

</Tabs>

### Verifying pacts where the consumer is a mobile application

Verify the pacts for the latest `master` and `test` versions, and all `production` versions of "my-mobile-consumer".

<Tabs
groupId="sdk-choice"
defaultValue="javascript"
values={[
{label: 'Javascript', value: 'javascript', },
{label: 'Java', value: 'java', },
{label: 'Ruby', value: 'ruby', },
{label: 'Python', value: 'python', },
{label: 'C#', value: 'c#', },
{label: 'Golang', value: 'golang', },
]
}>
<TabItem value="javascript">

```js
const verificationOptions = {
  // ...
  consumerVersionSelectors: [
    {
      tag: "master",
      latest: true,
    },
    {
      tag: "test",
      latest: true,
    },
    {
      tag: "production",
    },
  ],
};
```

  </TabItem>

  <TabItem value="java">

```java
@PactBroker(
host="pactbroker.local",
port="8080",
consumerVersionSelectors={
  @VersionSelector(tag = "master"),
  @VersionSelector(tag = "test"),
  @VersionSelector(tag = "production", latest = false)
})

```

  </TabItem>

  <TabItem value="ruby">

```ruby
Pact.service_provider "Your provider" do
  honours_pacts_from_pact_broker do
    pact_broker_base_url "..."
    consumer_version_selectors [
        { tag: "main", latest: true },
        { tag: "test", latest: true },
        { tag: "production" }
      ]
  end
end
```

  </TabItem>

  <TabItem value="python">

```python
verifier = Verifier(
    provider="Your provider",
    broker_url="...",
    consumer_version_selectors=[
        {"tag": "main", "latest": True},
        {"tag": "test", "latest": True},
        {"tag": "production"},
    ],
    # ...
)
```

  </TabItem>

<TabItem value="c#">

```csharp
      IPactVerifier pactVerifier = new PactVerifier(new PactVerifierConfig())
      .ProviderState($"providerServiceBaseUri/provider-states")
      .ServiceProvider("ProviderPacticipantName", "providerServiceBaseUri")
      .HonoursPactWith("ConsumerPacticipantName")
      .PactBroker(
        "http://pact-broker",
        consumerVersionSelectors: new List<VersionTagSelector>
        {
          new VersionTagSelector("master", latest: true),
          new VersionTagSelector("test", latest: true),
          new VersionTagSelector("production")
        });
```

  </TabItem>

  <TabItem value="golang">

```golang
 pact.VerifyProvider(t, provider.VerifyRequest{
  ConsumerVersionSelectors: []provider.Selector{
   &provider.ConsumerVersionSelector{
    Tag:         os.Getenv("GIT_BRANCH"),
    FallbackTag: "master",
    Latest:      true,
   },
   &provider.ConsumerVersionSelector{
    Tag:    "test",
    Latest: true,
   },
   &provider.ConsumerVersionSelector{
    Tag:    "production",
    Latest: false,
   },
  },
        // ...
 })
```

  </TabItem>

</Tabs>

### Verifying a pacts where one consumer is a mobile application

Verify the latest `production` version of all consumers, and all `production` versions of "my-mobile-consumer". Note that the pacts are [deduplicated](#deduplication), so despite being included by 2 selectors, the verification of the latest production pact for "my-mobile-consumer" will only run once.

<Tabs
groupId="sdk-choice"
defaultValue="javascript"
values={[
{label: 'Javascript', value: 'javascript', },
{label: 'Java', value: 'java', },
{label: 'Ruby', value: 'ruby', },
{label: 'Python', value: 'python', },
{label: 'C#', value: 'c#', },
{label: 'Golang', value: 'golang', }
]
}>
<TabItem value="javascript">

```js
const verificationOptions = {
  // ...
  consumerVersionSelectors: [
    {
      tag: "master",
      latest: true,
    },
    {
      tag: "test",
      latest: true,
    },
    {
      tag: "production",
      latest: true,
    },
    {
      tag: "production",
      consumer: "my-mobile-consumer",
    },
  ],
};
```

  </TabItem>

  <TabItem value="java">

```java
// Requires Pact-JVM 4.1.8 or later
@PactBroker(
  host="pactbroker.local",
  port="8080",
  consumerVersionSelectors={
    @VersionSelector(tag = "master"),
    @VersionSelector(tag = "test"),
    @VersionSelector(tag = "production"),
    @VersionSelector(tag = "production", consumer = "my-mobile-consumer")
  }
)

```

  </TabItem>

  <TabItem value="ruby">

```ruby
Pact.service_provider "Your provider" do
  honours_pacts_from_pact_broker do
    pact_broker_base_url "..."
    consumer_version_selectors [
        { tag: "main", latest: true },
        { tag: "test", latest: true },
        { tag: "production", latest: true },
        { tag: "production", consumer: "my-mobile-consumer" }
      ]
  end
end
```

  </TabItem>

  <TabItem value="python">

```python
verifier = Verifier(
    provider="Your provider",
    broker_url="...",
    consumer_version_selectors=[
        {"tag": "main", "latest": True},
        {"tag": "test", "latest": True},
        {"tag": "production", "latest": True},
        {"tag": "production", "consumer": "my-mobile-consumer"},
    ],
    # ...
)
```

  </TabItem>

  <TabItem value="c#">

```csharp
      IPactVerifier pactVerifier = new PactVerifier(new PactVerifierConfig())
      .ProviderState($"providerServiceBaseUri/provider-states")
      .ServiceProvider("ProviderPacticipantName", "providerServiceBaseUri")
      .HonoursPactWith("ConsumerPacticipantName")
      .PactBroker(
        "http://pact-broker",
        consumerVersionSelectors: new List<VersionTagSelector>
        {
          new VersionTagSelector("master", latest: true),
          new VersionTagSelector("test", latest: true),
          new VersionTagSelector("production", latest: true),
          new VersionTagSelector("production", "my-mobile-consumer")
        });
```

  </TabItem>

  <TabItem value="golang">

```golang
 pact.VerifyProvider(t, provider.VerifyRequest{
  ConsumerVersionSelectors: []provider.Selector{
   &provider.ConsumerVersionSelector{
    Tag:         os.Getenv("GIT_BRANCH"),
    FallbackTag: "master",
    Latest:      true,
   },
   &provider.ConsumerVersionSelector{
    Tag:         "test",
    Latest:      true,
   },
   &provider.ConsumerVersionSelector{
    Tag:         "production",
    Latest:      true,
   },
   &provider.ConsumerVersionSelector{
    Tag:         "production",
    Consumer:    "my-mobile-consumer",
   },
  },
        // ...
 })
```

  </TabItem>
</Tabs>

### Verifying the overall latest pact for each consumer

This is syntactically possible, but not recommended, as pacts for different branches of the consumer may overwrite each other as the current latest.

<Tabs
groupId="sdk-choice"
defaultValue="javascript"
values={[
{label: 'Javascript', value: 'javascript', },
{label: 'Java', value: 'java', },
{label: 'Ruby', value: 'ruby', },
{label: 'Python', value: 'python', },
{label: 'C#', value: 'c#', },
{label: 'Golang', value: 'golang', }
]
}>
<TabItem value="javascript">

```js
const verificationOptions = {
  // ...
  consumerVersionSelectors: [
    {
      latest: true,
    },
  ],
};
```

  </TabItem>

  <TabItem value="java">

```java
// Requires Pact-JVM 4.1.8 or later
@PactBroker(
  host="pactbroker.local",
  port="8080",
  consumerVersionSelectors={ @VersionSelector(latest = "true") }
)

```

  </TabItem>

  <TabItem value="ruby">

```ruby
Pact.service_provider "Your provider" do
  honours_pacts_from_pact_broker do
    pact_broker_base_url "..."
    consumer_version_selectors [
        { latest: true }
      ]
  end
end
```

  </TabItem>

  <TabItem value="python">

```python
verifier = Verifier(
    provider="Your provider",
    broker_url="...",
    consumer_version_selectors=[
        {"latest": True},
    ],
    # ...
)
```

  </TabItem>

  <TabItem value="c#">

```csharp
      IPactVerifier pactVerifier = new PactVerifier(new PactVerifierConfig())
      .ProviderState($"providerServiceBaseUri/provider-states")
      .ServiceProvider("ProviderPacticipantName", "providerServiceBaseUri")
      .HonoursPactWith("ConsumerPacticipantName")
      .PactBroker(
        "http://pact-broker",
        consumerVersionSelectors: new List<VersionTagSelector>
        {
          new VersionTagSelector("", latest: true)
        });
```

  </TabItem>

  <TabItem value="golang">

```golang
 pact.VerifyProvider(t, provider.VerifyRequest{
  ConsumerVersionSelectors: []provider.Selector{
   &provider.ConsumerVersionSelector{
    Latest:      true,
   },
  },
        // ...
 })
```

  </TabItem>

</Tabs>
