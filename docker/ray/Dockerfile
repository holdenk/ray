ARG GPU
ARG DOCKER_PREFIX

FROM "$DOCKER_PREFIX"ray-wheel:nightly"$GPU" as raywheel

FROM "$DOCKER_PREFIX"ray-deps:nightly"$GPU"
WORKDIR /ray
# For Click
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8
# Specify bazel (see https://github.com/bazelbuild/bazel/issues/12887)
ENV USE_BAZEL_VERSION=3.7.2
# py-spy on arm is happier with rust install as per https://gist.github.com/heavyinfo/aa0bf2feb02aedb3b38eef203b4b8cc4
RUN sudo apt-get update && sudo apt-get install -y curl bash
RUN echo "export PATH=~/anaconda3/bin/:~/.cargo/bin:\$PATH" >> ~/.bashrc
SHELL ["/bin/bash", "-c"]
# Install py-spy, it runs out of process but has difficulty installing in ARM. To allow it to be installed from source on ARM we need to install cargo.
RUN export PATH=$HOME/.cargo/bin:$HOME/anaconda3/bin/:$PATH
RUN (mamba install --yes py-spy) ||  \
    (pushd /tmp && curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs -o rustup.sh && chmod a+x rustup.sh && ./rustup.sh -y && source $HOME/.cargo/env && git clone https://github.com/benfred/py-spy && pushd py-spy && python setup.py install && popd && popd)
COPY --from=raywheel /ray/python/dist/ray*.whl .
RUN pip --no-cache-dir install $(ls ray*.whl)[all]
