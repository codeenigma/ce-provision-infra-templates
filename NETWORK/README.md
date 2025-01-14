# Configuring CI
This repo comes with a working CI implementation, however it relies on the existence of a variable called `ENV` that defaults to `none` as a value. Ensure this exists either locally in the repo or, more commonly, globally for the entire GitLab instance, e.g.

* https://gitlab.controller.acme.com/admin/application_settings/ci_cd

# Infra repo
It is recommended you create a new Project (repository) with the pattern `infra-SHORTNAME`, e.g. `infra-main-website`, and the URL https://gitlab.controller.acme.com/infras/infra-main-website.

It is recommended you create two branches, `apply` for live code and `test` for development and linting, and remove all other branches. `apply` should be the default branch and `test` should be protected to enforce the use of merge requests for code changes.

Within that new repo you need to enable the main `controller` deploy key under 'Settings -> Repository -> Deploy keys' and also enable your runner under 'Settings -> CI/CD -> Runners'.

# Seeding network
This repo is a template for seeding the network. Copy it into the `apply` branch of the `infra-SHORTNAME` repo you created above to get started. Then you need to find and replace these strings before committing it to your new infra:

* `SHORTNAME` - should be your infra shortname, e.g. `main-website`
* `CIDR_BASE` - should be a CIDR base, e.g. `10.116`, but it should be unique to allow for potential future VPC peering

Tip - this terminal command will make your life easier: `find ./ -type f -exec sed -i -e 's/SHORTNAME/main-website/g' {} \;`

Note, this repo is set up to create networks in the Dublin region, `eu-west-1`. If you want to use another region you should update the `REGION` variable in `.gitlab-ci.yml` and rename the `vars/eu-west-1` directory to match your desired region. If you want to operate in multiple regions, add them to the list in the `REGION` variable in `.gitlab-ci.yml` and copy the `vars/eu-west-1` to create a directory for each region you need. E.g. if I want Dublin and Frankfurt, but CI defaulting to Dublin, my CI would look like this:

```yaml
  REGION:
    value: "eu-west-1"
    options:
      - "eu-west-1"
      - "eu-central-1"
    description: "AWS region to pass to Ansible when building an individual host."
```

And I would have two directories in `vars/_regions`, one for `eu-west-1` and another for `eu-central-1`.

# First build
Once you've committed your initial code and [created a GitLab Runner](https://docs.gitlab.com/ee/tutorials/create_register_first_runner/) ([see also ce-provision instructions for this](https://github.com/codeenigma/ce-provision/wiki/Infrastructure-repository)) you can go to `Build` -> `Pipelines` -> `Run Pipeline` to launch a network build. You will need to build the environments one at a time, usually starting with `util`.

# Enabling NAT gateways
We disable the creation of NAT gateways by default because they are very expensive, but if you have an autoscaling group or an ECS cluster in your environment then you will need NAT gateways on your subnets. To enable them find the `aws_vpc_subnet.yml` file in the region and environment concerned under `vars/_regions` and change `nat_ipv4: false` to `nat_ipv4: true`.
