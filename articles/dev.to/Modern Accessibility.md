## Guessing at What the State of the Art in Web Accessibility Will Look Like in 5 Years

Web design and development evolve at a rapid pace. It's reasonable to assume web accessibility will evolve similarly over the next few years.

This is my attempt to guess at what the best organizations will be doing for it in 5 years time (inspired by [the Modern Testing principles](https://www.moderntesting.org/) and the [DevOps Research and Assessment studies](https://www.devops-research.com/research.html)).

## Protecting Abled Usage and Optimizing for Experimentation with Disabled Experiences

There are 2 high-level aspects to my guess at what the top organizations will be doing

1. Protecting Abled (i.e. non-disabled) Experiences
2. Optimizing for Experimentation with Disabled Experiences 

### Protecting Abled Experiences

Before organizations can focus on optimizing their disabled users' experiences, they'll have to first make sure they avoid impacting or regressing their abled users' experiences.

#### Avoidance of Designs that Cannot Be Made Accessible Without Affecting Abled Experiences

At the design level, this means avoiding design patterns  that cannot be made accessible without changing how abled users experience them.

For example, take ephemeral toast notifications. There's no way to make these accessible after-the-fact without creating a drawer containing all of the notifications. If there's no room for such a drawer in the design without impacting the abled experience, the design can't be made accessible.

For a contrary example, take icon buttons that end up not having accessible labels. They can be given accessible labels without needing to adjust the abled experience at all. The design can be made accessible without impacting the abled experience.

#### Functional and Visual Regression Tests of Abled Workflows Derived From Telemetry

Automated functional tests protecting abled user flows can give teams experimenting with accessibility changes extra confidence that their changes won't break abled use cases.

Layering on automated visual regression tests can enhance that confidence to another level.

And deriving the tests from production telemetry makes sure that the right abled user flows are being encoded into automation.

### Optimizing for Experimentation with Disabled Experiences

All of that protection should enable teams to experiment at will when it comes to disabled user experiences, without fear of side-effects.

#### Lead Times on the Order of Seconds

A key prerequisite to this is having an extremely short time in-between having an idea for an experiment and getting real feedback on it.

Bringing down lead times to a few seconds helps to this end. Tests can be shifted to run as synthetic monitors against production to help with this.

#### Living in Production

At this point, teams will be effectively "living in production" and can focus on experimentation.

##### Experiment-first Mentality

At the end of the day, the only people who can tell if an experience is accessible are the disabled users who experience it. No amount of prior team experience or knowledge can substitute for this.

Teams will construct experiments on how to improve disabled user experiences and measure them through several means in production. The successful experiments will live on, and the team will continue to iterate via experiments.

##### Analytics-first Mentality

Teams will leverage anonymized, aggregated metrics derived from their analytics that serve as indicators for disabled user experiences.

A common tactic will be to compare metrics (e.g. conversion rates) for disabled user groups against abled user groups. If the disabled user groups are performing more poorly than their abled counterparts, it will indicate more experimentation is needed.

##### Zero-Effort Generation of Strong Ties to the Disability Community

Analytics alone won't generate useful enough information. Teams will leverage 3rd parties to connect them with their disabled users so they can apply user research techniques to better understand their disabled users and drive their future experiments.

## Summary

My guess is the organizations that will be doing the best at accessibility will be the ones that work the closest with their disabled users, and then optimize all of their processes towards experimenting to find the best solutions for those users.

(Slack already does parts of this today to my understanding, which is why it doesn't seem too farfetched to me.)

 