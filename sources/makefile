#
# Copyright 2019 Google LLC
#
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
# 
#   http://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.
#

#------------------------------------------------------------------------------
# Basic configuration
#------------------------------------------------------------------------------

PRODUCT_VERSION = 2.8
CONFIGURATION = Release
PLATFORM=x86

NUNIT_ASSEMBLIES = \
    Google.Solutions.Common.Test\bin\$(CONFIGURATION)\Google.Solutions.Common.Test.dll
#	\
#    Google.Solutions.IapTunneling.Test\bin\release\Google.Solutions.IapTunneling.Test.dll \
#    Google.Solutions.IapDesktop.Extensions.Activity.Test\bin\release\Google.Solutions.IapDesktop.Extensions.Activity.Test.dll \
#    Google.Solutions.IapDesktop.Extensions.Os.Test\bin\release\Google.Solutions.IapDesktop.Extensions.Os.Test.dll \
#	Google.Solutions.IapDesktop.Extensions.Rdp.Test\bin\release\Google.Solutions.IapDesktop.Extensions.Rdp.Test.dll \
#    Google.Solutions.IapDesktop.Application.Test\bin\release\Google.Solutions.IapDesktop.Application.Test.dll

NUNIT_COVERAGE_FILTER = \
	+[Google.Solutions.Common]* \
	+[Google.Solutions.IapTunneling]* \
	+[Google.Solutions.IapDesktop.Extensions.Activity]* \
	+[Google.Solutions.IapDesktop.Extensions.Os]* \
	+[Google.Solutions.IapDesktop.Extensions.Rdp]* \
	+[Google.Solutions.IapDesktop.Application]*
#------------------------------------------------------------------------------
# Ancillary variables
#------------------------------------------------------------------------------

!if ( "$(__BUILD_ENV_INITIALIZED)" == "" ) 
!	ERROR "build must be invoked by build.ps1"
!endif

!if ( "$(KOKORO_BUILD_NUMBER)" != "" ) 
BINARY_VERSION = $(PRODUCT_VERSION).$(KOKORO_BUILD_NUMBER)
!else
BINARY_VERSION = $(PRODUCT_VERSION).0
!endif

POWERSHELL = powershell -NoProfile -ExecutionPolicy Unrestricted

#------------------------------------------------------------------------------
# Main targets
#------------------------------------------------------------------------------

default: kokoro-continuous-integration

cmd:
	@echo "========================================================"
	@echo "=== Opening command prompt                           ==="
	@echo "========================================================"

	cmd

clean:
	@echo "========================================================"
	@echo "=== Cleaning up                                      ==="
	@echo "========================================================"

	$(POWERSHELL) $(MAKEDIR)\scripts\clean.ps1


check-copyright-headers:
	@echo "========================================================"
	@echo "=== Checking copyright headers                       ==="
	@echo "========================================================"

	$(POWERSHELL) $(MAKEDIR)\scripts\check-copyright-headers.ps1


Google.Solutions.IapDesktop\OAuthClient.cs:
	@echo "========================================================"
	@echo "=== Patching OAuth credentials                      ==="
	@echo "========================================================"

	copy "$(KOKORO_GFILE_DIR)\OAuthClient.cs" $@

restore:
	@echo "========================================================"
	@echo "=== Restoring nuget packages                         ==="
	@echo "========================================================"

	nuget restore

build: Google.Solutions.IapDesktop\OAuthClient.cs restore
	@echo "========================================================"
	@echo "=== Building solution                                ==="
	@echo "========================================================"

	msbuild /t:Rebuild /p:Configuration=$(CONFIGURATION);Platform=$(PLATFORM);AssemblyVersionNumber=$(BINARY_VERSION)

test: $(NUNIT_ASSEMBLIES)
	@echo "========================================================"
	@echo "=== Testing solution                                 ==="
	@echo "========================================================"

	OpenCover.Console.exe \
		-register:user \
		-returntargetcode \
		-target:nunit3-console.exe \
		"-targetargs:$(NUNIT_ASSEMBLIES) --result=sponge_log.xml;transform=..\kokoro\nunit-to-sponge.xsl" \
		-filter:"$(NUNIT_COVERAGE_FILTER)" \
		"-excludebyattribute:*.SkipCodeCoverage*;*CompilerGenerated*" \
		-output:opencovertests.xml

	ReportGenerator.exe \
		-reports:opencovertests.xml \
		-targetdir:coveragereport \
		-reporttypes:HTML
	
installer: build
	@echo "========================================================"
	@echo "=== Building installer                               ==="
	@echo "========================================================"

	$(POWERSHELL) .\build-installer.ps1 \
		-ProductVersion $(PRODUCT_VERSION) \
		-Configuration $(CONFIGURATION)

#------------------------------------------------------------------------------
# Kokoro specific targets
#------------------------------------------------------------------------------

kokoro-iap-firewallrule:
	@echo "========================================================"
	@echo "=== Creating firewall rule                           ==="
	@echo "========================================================"

	gcloud auth activate-service-account --key-file=$(GOOGLE_APPLICATION_CREDENTIALS)
	gcloud compute firewall-rules create allow-ingress-from-iap \
		--direction=INGRESS \
		--action=allow \
		--rules=tcp \
		--source-ranges=35.235.240.0/20 \
		--project=$(GOOGLE_CLOUD_PROJECT)

kokoro-continuous-integration: kokoro-iap-firewallrule installer test